---
description: List active channels
---

# PUBSUB CHANNELS

## Syntax

    PUBSUB CHANNELS [pattern]

**Time complexity:** O(N) where N is the number of active channels, and assuming constant time pattern matching (relatively short channels and patterns)

Lists the currently *active channels*, whose name matches `pattern`.

An active channel is a Pub/Sub channel with one or more subscribers (excluding clients subscribed to patterns).

If no `pattern` is specified, all the channels are listed, otherwise if pattern is specified only channels matching the specified glob-style pattern are listed.

## Return

[Array reply](https://redis.io/docs/reference/protocol-spec#resp-arrays): a list of active channels, optionally matching the specified pattern.

## Examples

```shell
users> PSUBSCRIBE h?llo
users> SUBSCRIBE hello
users> SUBSCRIBE hxxllo
users> SUBSCRIBE world
---
dragonfly> PUBSUB CHANNELS
1) "hxxllo"
2) "world"
3) "hello"
dragonfly> PUBSUB CHANNELS h*llo
1) "hxxllo"
2) "hello"
dragonfly> PUBSUB CHANNELS h?llo
1) "hello"
```

