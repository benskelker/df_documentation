---
description: Set the string value and the ttl of a hash field
---

# HSETEX

## Syntax

    HSETEX key seconds field value [field value ...]

**Time complexity:** O(1) for each field/value pair added, so O(N) to add N field/value pairs when the command is called with multiple field/value pairs.

**Warning:** Experimental! Dragonfly specific.

Simimar to HSET but adds one or more hash fieds that expire after specified number of seconds.
This command overwrites the values of specified fields that exist in the hash.
If `key` doesn't exist, a new key holding a hash is created. In any case, the ttl is
updated according to the latest value and the current clock.

## Return

[Integer reply](https://redis.io/docs/reference/protocol-spec#resp-integers): The number of fields that were added.

## Examples

```shell
dragonfly> HSETEX myhash 5 field1 "Hello"
(integer) 1
# wait for 4 seconds
dragonfly> HGETALL myhash
1) "field1"
2) "Hello"
# wait for 1 seconds
dragonfly> HGETALL myhash
(empty array)
```
