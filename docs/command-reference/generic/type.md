---
description: Determine the type stored at key
---

# TYPE

## Syntax

    TYPE key

**Time complexity:** O(1)

Returns the string representation of the type of the value stored at `key`.
The different types that can be returned are: `string`, `list`, `set`, `zset`,
`hash`, `stream` and `ReJSON-RL`.

## Return

[Simple string reply](https://redis.io/docs/reference/protocol-spec#resp-simple-strings): type of `key`, or `none` when `key` does not exist.

## Examples

```shell
dragonfly> SET key1 "value"
"OK"
dragonfly> LPUSH key2 "value"
(integer) 1
dragonfly> SADD key3 "value"
(integer) 1
dragonfly> TYPE key1
"string"
dragonfly> TYPE key2
"list"
dragonfly> TYPE key3
"set"
```
