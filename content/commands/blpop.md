---
acl_categories:
- '@write'
- '@list'
- '@slow'
- '@blocking'
arguments:
- display_text: key
  key_spec_index: 0
  multiple: true
  name: key
  type: key
- display_text: timeout
  name: timeout
  type: double
arity: -3
categories:
- docs
- develop
- stack
- oss
- rs
- rc
- oss
- kubernetes
- clients
command_flags:
- write
- blocking
complexity: O(N) where N is the number of provided keys.
description: Removes and returns the first element in a list. Blocks until an element
  is available otherwise. Deletes the list if the last element was popped.
group: list
hidden: false
history:
- - 6.0.0
  - '`timeout` is interpreted as a double instead of an integer.'
key_specs:
- RW: true
  access: true
  begin_search:
    spec:
      index: 1
    type: index
  delete: true
  find_keys:
    spec:
      keystep: 1
      lastkey: -2
      limit: 0
    type: range
linkTitle: BLPOP
since: 2.0.0
summary: Removes and returns the first element in a list. Blocks until an element
  is available otherwise. Deletes the list if the last element was popped.
syntax_fmt: BLPOP key [key ...] timeout
syntax_str: timeout
title: BLPOP
---
`BLPOP` is a blocking list pop primitive.
It is the blocking version of [`LPOP`]({{< relref "/commands/lpop" >}}) because it blocks the connection when there
are no elements to pop from any of the given lists.
An element is popped from the head of the first list that is non-empty, with the
given keys being checked in the order that they are given.

## Non-blocking behavior

When `BLPOP` is called, if at least one of the specified keys contains a
non-empty list, an element is popped from the head of the list and returned to
the caller together with the `key` it was popped from.

Keys are checked in the order that they are given.
Let's say that the key `list1` doesn't exist and `list2` and `list3` hold
non-empty lists.
Consider the following command:

```
BLPOP list1 list2 list3 0
```

`BLPOP` guarantees to return an element from the list stored at `list2` (since
it is the first non empty list when checking `list1`, `list2` and `list3` in
that order).

## Blocking behavior

If none of the specified keys exist, `BLPOP` blocks the connection until another
client performs an [`LPUSH`]({{< relref "/commands/lpush" >}}) or [`RPUSH`]({{< relref "/commands/rpush" >}}) operation against one of the keys.

Once new data is present on one of the lists, the client returns with the name
of the key unblocking it and the popped value.

When `BLPOP` causes a client to block and a non-zero timeout is specified,
the client will unblock returning a `nil` multi-bulk value when the specified
timeout has expired without a push operation against at least one of the
specified keys.

**The timeout argument is interpreted as a double value specifying the maximum number of seconds to block**. A timeout of zero can be used to block indefinitely.

## What key is served first? What client? What element? Priority ordering details.

* If the client tries to blocks for multiple keys, but at least one key contains elements, the returned key / element pair is the first key from left to right that has one or more elements. In this case the client is not blocked. So for instance `BLPOP key1 key2 key3 key4 0`, assuming that both `key2` and `key4` are non-empty, will always return an element from `key2`.
* If multiple clients are blocked for the same key, the first client to be served is the one that was waiting for more time (the first that blocked for the key). Once a client is unblocked it does not retain any priority, when it blocks again with the next call to `BLPOP` it will be served accordingly to the number of clients already blocked for the same key, that will all be served before it (from the first to the last that blocked).
* When a client is blocking for multiple keys at the same time, and elements are available at the same time in multiple keys (because of a transaction or a Lua script added elements to multiple lists), the client will be unblocked using the first key that received a push operation (assuming it has enough elements to serve our client, as there may be other clients as well waiting for this key). Basically after the execution of every command Redis will run a list of all the keys that received data AND that have at least a client blocked. The list is ordered by new element arrival time, from the first key that received data to the last. For every key processed, Redis will serve all the clients waiting for that key in a FIFO fashion, as long as there are elements in this key. When the key is empty or there are no longer clients waiting for this key, the next key that received new data in the previous command / transaction / script is processed, and so forth.

## Behavior of `BLPOP` when multiple elements are pushed inside a list.

There are times when a list can receive multiple elements in the context of the same conceptual command:

* Variadic push operations such as `LPUSH mylist a b c`.
* After an [`EXEC`]({{< relref "/commands/exec" >}}) of a [`MULTI`]({{< relref "/commands/multi" >}}) block with multiple push operations against the same list.
* Executing a Lua Script with Redis 2.6 or newer.

When multiple elements are pushed inside a list where there are clients blocking, the behavior is different for Redis 2.4 and Redis 2.6 or newer.

For Redis 2.6 what happens is that the command performing multiple pushes is executed, and *only after* the execution of the command the blocked clients are served. Consider this sequence of commands.

    Client A:   BLPOP foo 0
    Client B:   LPUSH foo a b c

If the above condition happens using a Redis 2.6 server or greater, Client **A** will be served with the `c` element, because after the [`LPUSH`]({{< relref "/commands/lpush" >}}) command the list contains `c,b,a`, so taking an element from the left means to return `c`.

Instead Redis 2.4 works in a different way: clients are served *in the context* of the push operation, so as long as `LPUSH foo a b c` starts pushing the first element to the list, it will be delivered to the Client **A**, that will receive `a` (the first element pushed).

The behavior of Redis 2.4 creates a lot of problems when replicating or persisting data into the AOF file, so the much more generic and semantically simpler behavior was introduced into Redis 2.6 to prevent problems.

Note that for the same reason a Lua script or a `MULTI/EXEC` block may push elements into a list and afterward **delete the list**. In this case the blocked clients will not be served at all and will continue to be blocked as long as no data is present on the list after the execution of a single command, transaction, or script.

## `BLPOP` inside a `MULTI` / `EXEC` transaction

`BLPOP` can be used with pipelining (sending multiple commands and
reading the replies in batch), however this setup makes sense almost solely
when it is the last command of the pipeline.

Using `BLPOP` inside a [`MULTI`]({{< relref "/commands/multi" >}}) / [`EXEC`]({{< relref "/commands/exec" >}}) block does not make a lot of sense
as it would require blocking the entire server in order to execute the block
atomically, which in turn does not allow other clients to perform a push
operation. For this reason the behavior of `BLPOP` inside [`MULTI`]({{< relref "/commands/multi" >}}) / [`EXEC`]({{< relref "/commands/exec" >}}) when the list is empty is to return a `nil` multi-bulk reply, which is the same
thing that happens when the timeout is reached.

If you like science fiction, think of time flowing at infinite speed inside a
[`MULTI`]({{< relref "/commands/multi" >}}) / [`EXEC`]({{< relref "/commands/exec" >}}) block...

## Examples

```
redis> DEL list1 list2
(integer) 0
redis> RPUSH list1 a b c
(integer) 3
redis> BLPOP list1 list2 0
1) "list1"
2) "a"
```

## Reliable queues

When `BLPOP` returns an element to the client, it also removes the element from the list. This means that the element only exists in the context of the client: if the client crashes while processing the returned element, it is lost forever.

This can be a problem with some application where we want a more reliable messaging system. When this is the case, please check the [`BRPOPLPUSH`]({{< relref "/commands/brpoplpush" >}}) command, that is a variant of `BLPOP` that adds the returned element to a target list before returning it to the client.

## Pattern: Event notification

Using blocking list operations it is possible to mount different blocking
primitives.
For instance for some application you may need to block waiting for elements
into a Redis Set, so that as far as a new element is added to the Set, it is
possible to retrieve it without resort to polling.
This would require a blocking version of [`SPOP`]({{< relref "/commands/spop" >}}) that is not available, but using
blocking list operations we can easily accomplish this task.

The consumer will do:

```
LOOP forever
    WHILE SPOP(key) returns elements
        ... process elements ...
    END
    BRPOP helper_key
END
```

While in the producer side we'll use simply:

```
MULTI
SADD key element
LPUSH helper_key x
EXEC
```

## Return information

{{< multitabs id="blpop-return-info" 
    tab1="RESP2" 
    tab2="RESP3" >}}

One of the following:
* [Nil reply](../../develop/reference/protocol-spec#bulk-strings): no element could be popped and the timeout expired
* [Array reply](../../develop/reference/protocol-spec#arrays): the key from which the element was popped and the value of the popped element.

-tab-sep-

One of the following:
* [Null reply](../../develop/reference/protocol-spec#nulls): no element could be popped and the timeout expired
* [Array reply](../../develop/reference/protocol-spec#arrays): the key from which the element was popped and the value of the popped element.

{{< /multitabs >}}
