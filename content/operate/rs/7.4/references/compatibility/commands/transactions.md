---
Title: Transaction commands compatibility
alwaysopen: false
categories:
- docs
- operate
- rs
- rc
description: Transaction commands compatibility.
linkTitle: Transactions
weight: 10
url: '/operate/rs/7.4/references/compatibility/commands/transactions/'
---

The following table shows which Redis Open Source [transaction commands]({{< relref "/commands" >}}?group=transactions) are compatible with standard and Active-Active databases in Redis Enterprise Software and Redis Cloud.

| Command | Redis<br />Enterprise | Redis<br />Cloud | Notes |
|:--------|:----------------------|:-----------------|:------|
| [DISCARD]({{< relref "/commands/discard" >}}) | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> |  |
| [EXEC]({{< relref "/commands/exec" >}}) | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> |  |
| [MULTI]({{< relref "/commands/multi" >}}) | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> |  |
| [UNWATCH]({{< relref "/commands/unwatch" >}}) | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> |  |
| [WATCH]({{< relref "/commands/watch" >}}) | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> | <span title="Supported">&#x2705; Standard</span><br /><span title="Supported"><nobr>&#x2705; Active-Active</nobr></span> |  |
