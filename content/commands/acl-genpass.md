---
acl_categories:
- '@slow'
arguments:
- display_text: bits
  name: bits
  optional: true
  type: integer
arity: -2
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
- noscript
- loading
- stale
complexity: O(1)
description: Generates a pseudorandom, secure password that can be used to identify
  ACL users.
group: server
hidden: false
linkTitle: ACL GENPASS
since: 6.0.0
summary: Generates a pseudorandom, secure password that can be used to identify ACL
  users.
syntax_fmt: ACL GENPASS [bits]
syntax_str: ''
title: ACL GENPASS
---
ACL users need a solid password in order to authenticate to the server without
security risks. Such password does not need to be remembered by humans, but
only by computers, so it can be very long and strong (unguessable by an
external attacker). The `ACL GENPASS` command generates a password starting
from /dev/urandom if available, otherwise (in systems without /dev/urandom) it
uses a weaker system that is likely still better than picking a weak password
by hand.

By default (if /dev/urandom is available) the password is strong and
can be used for other uses in the context of a Redis application, for
instance in order to create unique session identifiers or other kind of
unguessable and not colliding IDs. The password generation is also very cheap
because we don't really ask /dev/urandom for bits at every execution. At
startup Redis creates a seed using /dev/urandom, then it will use SHA256
in counter mode, with HMAC-SHA256(seed,counter) as primitive, in order to
create more random bytes as needed. This means that the application developer
should be feel free to abuse `ACL GENPASS` to create as many secure
pseudorandom strings as needed.

The command output is a hexadecimal representation of a binary string.
By default it emits 256 bits (so 64 hex characters). The user can provide
an argument in form of number of bits to emit from 1 to 1024 to change
the output length. Note that the number of bits provided is always
rounded to the next multiple of 4. So for instance asking for just 1
bit password will result in 4 bits to be emitted, in the form of a single
hex character.

## Examples

```
> ACL GENPASS
"dd721260bfe1b3d9601e7fbab36de6d04e2e67b0ef1c53de59d45950db0dd3cc"

> ACL GENPASS 32
"355ef3dd"

> ACL GENPASS 5
"90"
```

## Return information

{{< multitabs id="acl-genpass-return-info" 
    tab1="RESP2" 
    tab2="RESP3" >}}

[Bulk string reply](../../develop/reference/protocol-spec#bulk-strings): pseudorandom data. By default it contains 64 bytes, representing 256 bits of data. If `bits` was given, the output string length is the number of specified bits (rounded to the next multiple of 4) divided by 4.

-tab-sep-

[Bulk string reply](../../develop/reference/protocol-spec#bulk-strings): pseudorandom data. By default it contains 64 bytes, representing 256 bits of data. If `bits` was given, the output string length is the number of specified bits (rounded to the next multiple of 4) divided by 4.

{{< /multitabs >}}
