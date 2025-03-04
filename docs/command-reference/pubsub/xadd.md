---
description: Appends a new entry to a stream
---

# XADD

## Syntax

    XADD key [NOMKSTREAM] [<MAXLEN | MINID> [= | ~] threshold [LIMIT count]] <* | id> field value [field value ...]

**Time complexity:** O(1) when adding a new entry, O(N) when trimming where N being the number of entries evicted.

Appends the specified stream entry to the stream at the specified key.
If the key does not exist, as a side effect of running this command the
key is created with a stream value. The creation of stream's key can be
disabled with the `NOMKSTREAM` option.

An entry is composed of a list of field-value pairs.
The field-value pairs are stored in the same order they are given by the user.
Commands that read the stream, such as `XRANGE` or `XREAD`, are guaranteed to return the fields and values exactly in the same order they were added by `XADD`.

`XADD` is the *only command* that can add data to a stream, but 
there are other commands, such as `XDEL` and `XTRIM`, that are able to
remove data from a stream.

## Specifying a Stream ID as an argument

A stream entry ID identifies a given entry inside a stream.

The `XADD` command will auto-generate a unique ID for you if the ID argument
specified is the `*` character (asterisk ASCII character). However, while
useful only in very rare cases, it is possible to specify a well-formed ID, so
that the new entry will be added exactly with the specified ID.

IDs are specified by two numbers separated by a `-` character:

    1526919030474-55

Both quantities are 64-bit numbers. When an ID is auto-generated, the
first part is the Unix time in milliseconds of the Dragonfly instance generating
the ID. The second part is just a sequence number and is used in order to
distinguish IDs generated in the same millisecond.

You can also specify an incomplete ID, that consists only of the milliseconds part, which is interpreted as a zero value for sequence part.
To have only the sequence part automatically generated, specify the milliseconds part followed by the `-` separator and the `*` character:

```shell
dragonfly> XADD mystream 1526919030474-55 message "Hello,"
"1526919030474-55"
dragonfly> XADD mystream 1526919030474-* message " World!"
"1526919030474-56"
```

IDs are guaranteed to be always incremental: If you compare the ID of the
entry just inserted it will be greater than any other past ID, so entries
are totally ordered inside a stream. In order to guarantee this property,
if the current top ID in the stream has a time greater than the current
local time of the instance, the top entry time will be used instead, and
the sequence part of the ID incremented. This may happen when, for instance,
the local clock jumps backward, or if after a failover the new master has
a different absolute time.

When a user specified an explicit ID to `XADD`, the minimum valid ID is
`0-1`, and the user *must* specify an ID which is greater than any other
ID currently inside the stream, otherwise the command will fail and return an error. Usually
resorting to specific IDs is useful only if you have another system generating
unique IDs (for instance an SQL table) and you really want the Dragonfly stream
IDs to match the one of this other system.

## Capped streams

`XADD` incorporates the same semantics as the `XTRIM` command - refer to its documentation page for more information.
This allows adding new entries and keeping the stream's size in check with a single call to `XADD`, effectively capping the stream with an arbitrary threshold.
Although exact trimming is possible and is the default, due to the internal representation of steams it is more efficient to add an entry and trim stream with `XADD` using **almost exact** trimming (the `~` argument).

For example, calling `XADD` in the following form:

    XADD mystream MAXLEN ~ 1000 * ... entry fields here ...
 
Will add a new entry but will also evict old entries so that the stream will contain only 1000 entries, or at most a few tens more.

## Return

[Bulk string reply](https://redis.io/docs/reference/protocol-spec#resp-bulk-strings), specifically:

The command returns the ID of the added entry. The ID is the one auto-generated
if `*` is passed as ID argument, otherwise the command just returns the same ID
specified by the user during insertion.

The command returns a [Null reply](https://redis.io/docs/reference/protocol-spec#resp-bulk-strings) when used with the `NOMKSTREAM` option and the
key doesn't exist.

## Examples

```shell
dragonfly> XADD mystream * name Sara surname OConnor
"1676903939979-0"
dragonfly> XADD mystream * field1 value1 field2 value2 field3 value3
"1676903939979-1"
dragonfly> XLEN mystream
(integer) 2
dragonfly> XRANGE mystream - +
1) 1) "1676903939979-0"
2) 1) "name"
2) "Sara"
3) "surname"
4) "OConnor"
2) 1) "1676903939979-1"
2) 1) "field1"
2) "value1"
3) "field2"
4) "value2"
5) "field3"
6) "value3"
```
