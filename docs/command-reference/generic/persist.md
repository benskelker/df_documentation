---
description: Remove the expiration from a key
---

# PERSIST

## Syntax

    PERSIST key

**Time complexity:** O(1)

Remove the existing timeout on `key`, turning the key from _volatile_ (a key
with an expire set) to _persistent_ (a key that will never expire as no timeout
is associated).

## Return

[Integer reply](https://redis.io/docs/reference/protocol-spec#resp-integers), specifically:

* `1` if the timeout was removed.
* `0` if `key` does not exist or does not have an associated timeout.

## Examples

```shell
dragonfly> SET mykey "Hello"
"OK"
dragonfly> EXPIRE mykey 10
(integer) 1
dragonfly> TTL mykey
(integer) 10
dragonfly> PERSIST mykey
(integer) 1
dragonfly> TTL mykey
(integer) -1
```
