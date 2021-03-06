---
navhome: /docs/
next: true
sort: 7
title: ~= "sigtis"
---

# `~= "sigtis"` 

`[%sgts p=hoon q=hoon]`: detect duplicate.

## Expands to

`q`.

## Convention

If `p` equals `q`, produce `p` instead of `q`.

## Discussion

Duplicates are especially bad news in Hoon, because comparing them
takes O(n) time.  Use `~=` ("sigtis") to kill them as they breed.

## Examples

This code traverses a tree and replaces all instances of `42` with
`420`:

```
~zod:dojo> =foo |=  a=(tree) 
                ?~(a ~ ~=(a [?:(=(n.a 42) 420 n.a) $(a l.a) $(a r.a)]))
~zod:dojo> (foo 42 ~ ~)
[420 ~ ~]
```

Without `~=`, it would build a copy of a completely unchanged tree.  Sad!
