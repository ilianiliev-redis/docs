---
aliases:
- /operate/oss_and_stack/install/install-redis/install-redis-on-mac-os
categories:
- docs
- operate
- stack
- oss
description: Use Homebrew to install and start Redis on macOS
linkTitle: MacOS
title: Install Redis on macOS
weight: 1
---

This guide shows you how to install Redis on macOS using Homebrew. Homebrew is the easiest way to install Redis on macOS. If you'd prefer to build Redis from the source files on macOS, see [Installing Redis from Source]({{< relref "/operate/oss_and_stack/install/archive/install-redis/install-redis-from-source" >}}).

{{< note >}}
The Homebrew distribution of Redis Open Source is only supported on macOS.
{{< /note >}}

## Prerequisites

First, make sure you have Homebrew installed. From the terminal, run:

{{< highlight bash  >}}
brew --version
{{< / highlight >}}

If this command fails, you'll need to [follow the Homebrew installation instructions](https://brew.sh/).

## Installation

From the terminal, run:

{{< highlight bash  >}}
brew install redis
{{< / highlight >}}

This will install Redis on your system.

## Starting and stopping Redis in the foreground

To test your Redis installation, you can run the `redis-server` executable from the command line:

{{< highlight bash  >}}
redis-server
{{< / highlight >}}

If successful, you'll see the startup logs for Redis, and Redis will be running in the foreground.

To stop Redis, enter `Ctrl-C`.

### Starting and stopping Redis using launchd

As an alternative to running Redis in the foreground, you can also use `launchd` to start the process in the background:

{{< highlight bash  >}}
brew services start redis
{{< / highlight >}}

This launches Redis and restarts it at login. You can check the status of a `launchd` managed Redis by running the following:

{{< highlight bash  >}}
brew services info redis
{{< / highlight >}}

If the service is running, you'll see output like the following:

{{< highlight bash  >}}
redis (homebrew.mxcl.redis)
Running: ✔
Loaded: ✔
User: miranda
PID: 67975
{{< / highlight >}}

To stop the service, run:

{{< highlight bash  >}}
brew services stop redis
{{< / highlight >}}

## Connect to Redis

Once Redis is running, you can test it by running `redis-cli`:

{{< highlight bash  >}}
redis-cli
{{< / highlight >}}

Test the connection with the `ping` command:

{{< highlight bash  >}}
127.0.0.1:6379> ping
PONG
{{< / highlight >}}

You can also test that your Redis server is running using
[Redis Insight]({{< relref "/develop/tools/insight" >}}).

## Next steps

Once you have a running Redis instance, you may want to:

* Try the [Redis CLI tutorial]({{< relref "/develop/tools/cli" >}})
* Connect using one of the [Redis clients]({{< relref "/develop/clients" >}})
* [Install Redis "properly"]({{< relref "/operate/oss_and_stack/install/archive/install-redis#install-redis-properly" >}})
  for production use.
  
