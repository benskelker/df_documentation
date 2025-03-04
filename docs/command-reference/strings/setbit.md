---
description: Sets or clears the bit at offset in the string value stored at key
---

# SETBIT

## Syntax

    SETBIT key offset value

**Time complexity:** O(1)

Sets or clears the bit at _offset_ (zero-indexed) in the string value stored at _key_.

The bit is either set or cleared depending on _value_, which can be either 0 or 1. 

When _key_ does not exist, a new string value is created.
The string is grown to make sure it can hold a bit at _offset_.
The _offset_ argument is required to be greater than or equal to 0, and smaller
than 2<sup>32</sup> (this limits bitmaps to 512MB).
When the string at _key_ is grown, added bits are set to 0.

**Warning**: When setting the last possible bit (_offset_ equal to 2<sup>32</sup>-1), and the string value stored at _key_ does not holds a value (or holds a small string), the operation will take some time, as Dragonfly is required to allocate all memory leading to that bit. Subsequent calls will not have the performance penalty.

## Return

[Integer reply](https://redis.io/docs/reference/protocol-spec#resp-integers): the original bit value stored at _offset_.

## Examples

```shell
dragonfly> SETBIT mykey 7 1
(integer) 0
dragonfly> SETBIT mykey 7 0
(integer) 1
dragonfly> GET mykey
" "
```

## Pattern: accessing the entire bitmap

There are cases when you need to set all the bits of single bitmap at once, for
example when initializing it to a default non-zero value. It is possible to do
this with multiple calls to the `SETBIT` command, one for each bit that needs to
be set. However, so as an optimization you can use a single `SET` command to set
the entire bitmap.

Bitmaps are not an actual data type, but a set of bit-oriented operations
defined on the String type (for more information refer to the
[Bitmaps section of the Data Types Introduction page][ti]). This means that
bitmaps can be used with string commands, and most importantly with `SET` and
`GET`.

A bitmap is trivially encoded as a bytes
stream. The first byte of the string corresponds to offsets 0..7 of
the bitmap, the second byte to the 8..15 range, and so forth.

For example, after setting a few bits, getting the string value of the bitmap
would look like this:

```shell
dragonfly> SETBIT bitmapsarestrings 2 1
dragonfly> SETBIT bitmapsarestrings 3 1
dragonfly> SETBIT bitmapsarestrings 5 1
dragonfly> SETBIT bitmapsarestrings 10 1
dragonfly> SETBIT bitmapsarestrings 11 1
dragonfly> SETBIT bitmapsarestrings 14 1
dragonfly> GET bitmapsarestrings
"42"
```

By getting the string representation of a bitmap, the client can then parse the
response's bytes by extracting the bit values using native bit operations in its
native programming language. Symmetrically, it is also possible to set an entire
bitmap by performing the bits-to-bytes encoding in the client and calling `SET`
with the resultant string.

[ti]: https://redis.io/topics/data-types-intro#bitmaps

## Pattern: setting multiple bits

`SETBIT` excels at setting single bits, and can be called several times when
multiple bits need to be set. To optimize this operation you can replace
multiple `SETBIT` calls with a single call to the variadic `BITFIELD` command
and the use of fields of type `u1`.

For example, the example above could be replaced by:

```
BITFIELD bitsinabitmap SET u1 2 1 SET u1 3 1 SET u1 5 1 SET u1 10 1 SET u1 11 1 SET u1 14 1
```

