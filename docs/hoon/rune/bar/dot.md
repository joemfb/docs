---
navhome: /docs/
next: true
sort: 4
title: |. "bardot"
---

# `|. "bardot"`

`[%brdt p=hoon]`: form a trap, a core with one arm `$`.

## Expands to

```
|%  ++  $  p
--
```

## Syntax

Regular: *1-fixed*.

## Discussion

A trap is a deferred computation.

## Examples

A trivial trap:

```
~zod:dojo> =foo |.(42)
~zod:dojo> $:foo
42
~zod:dojo> (foo)
42
```

A more interesting trap:

```
~zod:dojo> =foo  =/  reps  10
                 =/  step  0
                 =/  outp  0
                 |.
                 ?:  =(step reps)
                   outp
                 $(outp (add outp 2), step +(step))
~zod:dojo> (foo)
20
```

Note that we can use `$()` to recurse back into the
trap, since it's a core with an `$` arm.

> `$()` expands to `%=($)` (["centis"](../../cen/tis)), 
> accepting a *jogging* body containing a list of changes 
> to the subject.
