---
navhome: '/docs'
next: True
sort: 3
title: Urbit glossary
---

# Urbit glossary

Urbit is renowned for its exotic terminology. Here's a simple overview
from the strange words in.

As Dijkstra put it: "The purpose of abstraction is not to be vague, but
to create a new semantic level in which one can be absolutely precise."

## Ships

An Urbit `ship` is a cryptographic identity, a human-memorable name, and
a packet routing address. Ships are classed by the number of bits in
their address:

    Size   Name    Parent  Object      Example
    -----  ------  ------  ------      -------
    2^8    galaxy  ~       supernode   ~zod
    2^16   star    galaxy  supernode   ~doznec
    2^32   planet  star    user        ~tasfyn-partyv
    2^64   moon    planet  device      ~sigsam-nimbot-tasfyn-partyv
    2^128  comet   ~       bot         ~racmus-mollen-fallyt-linpex--watres-sibbur-modlux-rinmex

Any ship can be called an `urbit`.

> A summary [here](http://urbit.org/posts/address-space/).

An Urbit identity is a string like `~firbyr-napbes`. It means nothing,
but it's easy to remember and say out loud. `~firbyr-napbes` is actually
just a 32-bit number, like an IP address, that we turn into a
human-memorable string.

Technically, an urbit is a secure digital identity that you own and
control with a cryptographic key, like a Bitcoin wallet. As in Bitcoin,
the supply of urbits is mathematically limited. This keeps the network
friendly, by making spam and abuse expensive.

An urbit name is just a number; smaller numbers make shorter names.
Shorter names are easier to remember, so they're more valuable. So
urbits are classified by the number of bits in their name. (A ship name
is just a scrambled base-256 representation of the number.)

A 32-bit urbit (like `~firbyr-napbes`) is called a "planet." A 16-bit
urbit (like `~pollev)` is a "star." An 8-bit ship (like `~mun`) is a
"galaxy." A planet is an identity for an independent, adult human. Stars
and galaxies are network infrastructure.

Each planet or star is launched by its "parent," the star or galaxy
whose number is its bottom half. So the planet `~firbyr-napbes`,
`0xdead.beef` or `3.735.928.559`, is the child of `~pollev`, `0xbeef` or
`48.879`, whose parent is `~mun`, `0xef`, `239`. The parent of `~mun`
and all galaxies is `~zod`, `0`.

### Nock

> Nock is a Turing-complete, non-lambda combinator interpreter.

-   `noun`: an `atom` or a `cell`
-   `atom`: any natural number
-   `cell`: any ordered pair of `noun`s
-   `subject`: a `noun` - the data against which a `formula` is
    evaluated
-   `formula`: a `noun` - a function at the Nock level
-   `product`: a `noun` - the result of evaluating a `formula` against a
    `subject`

*See [Nock definition](../../nock/definition/).*

### Hoon

> Hoon is a strict, higher-order typed functional language that compiles
> itself to Nock.

###### `span`: an inferred type

A `span` defines a set (finite or infinite) of `noun`s and ascribes some
semantics to it. There is no Hoon syntax for a `span`; it is always
produced as the inferred range of an expression (`twig`).

*See [basic types](../../hoon/basic/#-type-span-and-mold).*

###### `core`: a code-data `cell`

The code (`battery`) is the head, the data (`payload`) is the tail. All
code-data structures in normal languages (functions, objects, modules,
etc) become `core`s in Hoon.

-   `battery`: the code of a `core`, a tree of `arm`s
-   `payload`: the data in a `core`

*See [basic types](../../hoon/basic)*.

###### `arm`: a named, functionally-computed attribute of a `core`

The `twig` of each `arm` is compiled to a Nock formula, with the
enclosing `core` itself as the subject.

There are two kinds of `arm`s: `dry` and `wet`. Most arms are dry.

-   `dry`: normal, `++`, polymorphic by means of *variance*

For a `dry` `arm`, we ask, is the new `payload` compatible with the old
`payload` (against which the `core` was compiled)?

-   `wet`: unusual, `+-`, polymorphic by means of *genericity*

A `wet` `arm` uses the `twig` as a macro. We create a new type analysis
path, which works as if we expanded the callee with the caller's
context.

*See [advanced types](/hoon/advanced)*.

###### `metal`: a variance model

Every `core` has a `metal` which defines its *variance* model (ie, the
properties of the `span` of a compatible `core`). The default is `gold`
(invariant).

-   `gold`: *invariant*
-   `lead`: *bivariant*
-   `zinc`: *covariant*
-   `iron`: *contravariant*

*See [advanced types](/hoon/advanced)*.

###### `gate`: a function/lambda/closure

A `gate` is a `core` with one `arm`. To call a `gate` on an argument,
replace the `sample` (at tree address `6` in the `core`) with the
argument, and then compute the `arm`.

The `payload` of a `gate` has a shape of `{sample context}`.

-   `sample`: the argument tuple
-   `context`: the subject in which the `gate` was defined

*See [basic types](../../hoon/basic/#-core-p-span-q-map-term-span),
[`%-` or `:call`](../../hoon/twig/cen-call/hep-call/) (the `twig` for
calling a `gate`).*

###### `mold`: a type constructor / validator

A mold is an idempotent `gate` (function), accepting any `noun`, and
producing a range with a useful `span`.

-   `bunt`: the value a `mold` produces when normalizing its default
    sample
-   `icon`: the `span` of the `mold`'s range

*See [`mold` `twig`s](/hoon/twig/buc-mold/).*

###### `atom`

A Hoon `atom` span describes a Nock `atom`, with two additional pieces
of metadata: an `aura`, and an optional constant.

An `atom` span is `warm` or `cold` based on whether the constant exists.

-   `warm`: if the constant is `~` (null), any `atom` is in the `span`
-   `cold`: if the constant is `[~ atom]`, its only legal value is
    `atom`.

*See [basic types](../../hoon/basic/#-atom-p-term-q-unit-atom)*

###### `aura`: a soft `atom` type

`aura` is a name for an atom type. It represents the structure of an
`atom` in a string beginning with `@`. An aura may represent units,
print format, or other semantics. Its constraints on the value of an
atom aren't enforced in any way.

Two `aura`s are compatible if one is a prefix of the other. You can
change any `aura` into any other `aura` by casting through the empty
`aura`, `@`. The standard library has molds which alias many common
auras.

Some common auras and their aliases:

-   `term` (`@tas`): a symbol - an atomic ASCII string which obeys
    symbol rules: lowercase and digit only, infix hyphen ("kebab-case"),
    first character must be lowercase alphabetic.
-   `cord` (`@t`): UTF-8 text, least-significant-byte first
-   `char` (`@tD`): a character, a single unicode byte (for multi-byte
    characters and codepoints, see `@c`)

*See [basic types](../../hoon/basic/#-atom-p-term-q-unit-atom)*.

###### `twig`: a Hoon expression

A `twig` is a Hoon expression. Specifically, `twig` is the mold for the
noun that a Hoon source expression compiles to. A twig is always a cell.

A twig has one of two forms: `{twig twig}`, which produces the ordered
pair of the two twigs, or a cell `{stem bulb}`, where `stem` is an
atomic symbol.

-   `stem`: an atomic symbol (`@tas`) - the name of a `twig`.
-   `bulb`: The `mold` of the `twig`'s contents.

Most `twig`s have a *regular form*, beginning with a `sigil` which is
either a `keyword` or a `rune`. Some `twig`s also have a syntactic
*irregular form*; a few have *only* an *irregular form*.

-   `sigil`: a `keyword` or `rune` used to begin a `twig`
-   `keyword`: the value of a stem, prefixed with `:`.
-   `rune`: a pair of ASCII symbols used to begin a `twig`.

For example, the stem `%if` becomes the keyword
[`:if`](../../hoon/twig/wut-test/col-if/), with the rune `?:`. The first
symbol in a rune represents a family of related `twig`s. For example,
the [`?` family](../../hoon/twig/wut-test/) are all conditionals.

The keyword or rune forms are up to the programmer's choice. Most
existing code uses runes, but the keyword form makes the learning path
easier.

A regular twig has two syntactic forms, *tall* and *flat*:

-   `tall`: multiple lines, no parentheses, two or more spaces between
    tokens
-   `flat`: one line, parentheses, one space between tokens

`tall` twigs can contain `flat` ones, but not vice versa. All irregular
forms are `flat`.

*See [`twig` concept](../../hoon/concepts/#-twig-expression),
[expressions](../../hoon/twig/), and [syntax](../../hoon/syntax/)*.

###### `limb`: attribute or variable reference

To resolve a `limb` named "foo", the `subject` is searched depth-first,
head-first for either a `face` named "foo" or a `core` with an `arm` of
"foo". If a `face` is found, the result is a `leg`, if a `core` is
found, the result is the `product` of the `arm`.

-   `leg`: a subexpression, or subtree of the `subject`.
-   `wing`: a list of `limb`s, searched from right to left (`a.b` means
    `b` within `a`).

*See [Limbs and wings](../../hoon/twig/limb)*

###### `face`: a labeled subtree

Hoon has no scope or symbol-table; there is only the `subject`. To
"declare" a "variable" is to construct a new `subject`:
`[name=value old-subject]`.

A `face` is a `span` that wraps a name (`@tas`) around another `span`.

*See [advanced types](../../hoon/advanced/#-face-aliases-and-bridges)*.

###### `nest`: type compatibility test

`nest` is an internal function on two spans, which produces yes if the
set of nouns in the second span is provably a subset of the first.

*See [advanced types](../../hoon/advanced/)*.

###### `loobean`: a Hoon boolean

`0` (`%.y`) is *yes*, `1` (`%.n`) is *no*.

> Why? It's fresh, it's different, it's new. And it's annoying. And it
> keeps you on your toes. And it's also just intuitively right.

###### `mark`: a type definition

A mark is fundamentally a type definition, but accessible at the arvo level. Each mark is defined in the `/mar` directory. Some marks have conversion routines to other marks, and some have diff, patch, and merge algorithms. None of these are required for a mark to exist, however.

*See [network](../../arvo/network/)*.

###### `clam`: a type validator function

In Hoon, when types are called as functions, they serve as a validator function called a "clam" -- that is, a function whose domain is all nouns, and range is the given type. if a clam is passed a value of its own type, it produces that value. Otherwise, it produces the default value (aka the "bunt") of its type.

*See [network](../../arvo/network/)*.

### Dojo

###### `source`: a type of :dojo command

A source is just something that can be printed to your console or the result of some computation. Sources can be chained together.

*See [Shell](../../using/shell/)*.

###### `sink`: a type of :dojo command

A sink is an effect: a change to the filesystem, a network message, a change to your environment, or typed message to an app. Sinks cannot be chained, we can only produce one effect per command.

*See [Shell](../../using/shell/)*.

###### `generator`: a hoon script

Generators are simple hoon scripts loaded from the filesystem. They live in `gen/`. Generators start with a `+` (lus). E.g `+ls` is similar to Unix ls. Accepts a path.

*See [Shell](../../using/shell/)*.

###### `hood`: the system daemon

The hood is the system daemon. See `gen/hood` and `app/hood`. ~~Hood daemons~~ begin with `|` (bar). E.g. `|start` starts an app. Accepts an app name.

*See [Shell](../../using/shell/)*.

### Arvo

###### `gall`: a stateful server

`gall` apps are stateful servers, sort of like unix daemons. One familiar one is app/talk.hoon which is the source code for :talk the urbit messaging transport layer. And there's also app/dojo.hoon — that's your shell.
