+++
title = "Make ; ('mic')"
weight = 13
template = "doc.html"
aliases = ["docs/reference/hoon-expressions/rune/mic/"]
+++

Miscellaneous useful macros.

## Runes

### `;:` "miccol"

`[%mccl p=hoon q=(list hoon)]`: call a binary function as an n-ary function.

##### Expands to

**Pseudocode**: `a`, `b`, `c`, ... as elements of `q`:

Regular form:

```hoon
%-(p a %-(p b %-(p c ...)))
```

Irregular form:

```hoon
(p a (p b (p c ...)))
```

##### Desugaring

```hoon
|-
?~  q  !!
?~  t.q  !!
?~  t.t.q
  (p i.q i.t.q)
(p i.q $(q t.q))
```

##### Syntax

Regular: **1-fixed**, then **running**.

Irregular: `:(add a b c)` is `;:(add a b c)`.

##### Examples

```
~zod:dojo> (add 3 (add 4 5))
12

~zod:dojo> ;:(add 3 4 5)
12

~zod:dojo> :(add 3 4 5)
12
```

### `;<` "micgal"

`[%mcgl p=spec q=hoon r=hoon s=hoon]`: monadic do notation.

##### Syntax

Regular: **4-fixed**

```hoon
;<  mold=mold  bind=hoon  expr1=hoon  expr2=hoon
```

#### Semantics

A `;<` is for sequencing two computations, `expr1` and `expr2`, using a provided
implementation of monadic bind. This rune takes a gate `bind` which takes a mold
`mold` and produces an implementation of monadic bind.

##### Desugaring

```hoon
%+  (bind mold)
  expr1
|=  mold
expr2
```

##### Discussion

`;<` is much like Haskell `do` notation. You have a sequence of events you'd
like to run such that each past the first depends on the output of the previous
one. The output of the previous one may not be of the correct type to use as an
input to the next one, and so an adapter such as `+biff` is needed.

`;<` differs from [`;~`](#micsig) in that it takes a gate which takes a mold
that produces an implementation of monadic bind, rather than taking an
implementation of monadic bind directly.

`;<` can be used to glue a pipeline together to run an asynchronous function or
event. This can be helpful when deferring parts of a computation based on
external data.

We remark that you can switch binds in the middle of a sequence of `;<`.

##### Examples

[`+biff`](/docs/hoon/reference/stdlib/2a/#biff) is the unit monad's
implementation of monadic bind. That is to say, it takes a unit `a` and a gate
`b` that accepts a noun that produces a unit, and extracts the value from `a` to
pass as a sample to `b`. 

We illustrate the usage of `;<` with `+biff` with a `map` of atoms:
```
~zod:dojo> =m (my ~[[1 3] [2 2] [3 1]])
         > (~(get by m) 1)
         [~ 3]
```
A single usage of `;<` only serves to apply the binding function to the output
of `expr1`:
```
~zod:dojo> ;<  a=@  _biff  (~(get by m) 1)
           a
         3
```
Here we see the result of chaining them together:
```
~zod:dojo> ;<  a=@  _biff  (~(get by m) 1)
           ;<  b=@  _biff  (~(get by m) a)
           b
         1
```



### `;+` "miclus"

make a single XML node (Sail)

##### Produces

A `marl`, i.e., a list of `manx`.  A `manx` is a noun that represents a single XML node.

##### Syntax

**1-fixed**

```hoon
;+  p=hoon
```

`p` is a Hoon expression that produces a `manx`.

##### Discussion

tl;dr -- `;+` converts a `manx` to a `marl`.

`;+` is a Sail rune.  Sail is a part of Hoon used for creating and operating on nouns that represent XML nodes.  With the appropriate rendering pipeline, a Sail document can be used to generate a static website.

In Sail a single XML node is represented by a `manx`.  A single <code>&lt;p&gt;</code> node `manx` can be produced in the following way:

```
> ;p: This will be rendered as an XML node.
[[%p ~] [[%$ [%$ "This will be rendered as an XML node."] ~] ~] ~]
```

Sometimes what is needed is a `marl`, i.e., a list of `manx`.  To convert a single `manx` to a `marl`, use the `;+` rune.

One interesting thing about Sail is that it allows you to use complex Hoon expressions to choose from among several nodes to render.  The `;+` rune can take such a complex expression.

##### Examples

```
> ^-  marl
  ;+  ?:  (gth 3 2)
        ;p: This is the node for 'yes'.
      ;p: This is the node for 'no'.
~[
  [ g=[n=%p a=~]
    c=[i=[g=[n=%$ a=~[[n=%$ v="This is the node for 'yes'."]]] c=~] t=~]
  ]
]

> ^-  marl
  ;+  ?:  (gth 2 3)
        ;p: This is the node for 'yes'.
      ;p: This is the node for 'no'.
~[
  [ g=[n=%p a=~]
    c=[i=[g=[n=%$ a=~[[n=%$ v="This is the node for 'no'."]]] c=~] t=~]
  ]
]
```

### `;;` "micmic"

`[%mcmc p=spec q=hoon]`: normalize with a mold, asserting fixpoint.

##### Expands to

```hoon
=+  a=(p q)
?>  =(`*`a `*`q)
a
```

> Note: the expansion implementation is hygienic -- it doesn't actually add the `a` face to the subject.

##### Syntax

Regular: **2-fixed**.

##### Examples

Fails because of auras:

```
~zod:dojo> ^-(tape ~[97 98 99])
mint-nice
nest-fail
ford: %slim failed: 
ford: %ride failed to compute type:
```

Succeeds because molds don't care about auras:

```
~zod:dojo> ;;(tape ~[97 98 99])
"abc"
```

Fails because not a fixpoint:

```
~zod:dojo> ;;(tape [50 51 52])
ford: %ride failed to execute:
```

### `;/` "micfas"

`[%mcnt p=hoon]`: tape as XML element.

##### Expands to

```hoon
~[%$ ~[%$ 'p']]
```

##### Examples

```
~zod/try=> ;/  "foo"
[[%~. [%~. "foo] ~] ~]
```

```
~zod/try=> :/"foo"
[[%~. [%~. "foo] ~] ~]
```

### `;~` "micsig"

`[%mcsg p=hoon q=(list hoon)]`: glue a pipeline together with a
product-sample adapter.

##### Produces

The gates in `q` are composed together using the gate `p` as an intermediate function, which transforms a `q` product and a `q` gate into a `q` sample.

##### Expands to

**Note: these are structurally correct, but elide some type-system complexity.**

`;~(a b)` reduces to `b`.

`;~(a b c)` expands to

```hoon
|=  arg=*
(a (b arg) c(+6 arg))
```

`;~(a b c d)` expands to

```hoon
|=  arg=*
%+  a (b arg)
=+  arg=arg
|.  (a (c arg) d(+6 arg))
```

##### Desugaring

```hoon
?~  q  !!
|-
?~  t.q  i.q
=/  a  $(q t.q)
=/  b  i.q
=/  c  ,.+6.b
|.  (p (b c) a(,.+6 c))
```

##### Discussion

Apparently `;~` is a "Kleisli arrow."  It's also
a close cousin of the infamous "monad."  Don't let that bother
you.  Hoon doesn't know anything about category theory,
so you don't need to either.

`;~` is often used in parsers, but is not only for parsers.

This can be thought of as user-defined function composition; instead of simply
nesting the gates in `q`, each is passed individually to `p` with the product
of the previous gate, allowing arbitrary filtering, transformation, or
conditional application.

##### Syntax

Regular: **1-fixed**, then **running**.

##### Examples

A simple "parser."  `trip` converts a `cord` (atomic string) to
a `tape` (linked string).

```
~zod:dojo> =cmp |=([a=tape b=$-(char tape)] `tape`?~(a ~ (weld (b i.a) t.a)))
~zod:dojo> ;~(cmp trip)
<1.zje {a/@ <409.yxa 110.lxv 1.ztu $151>}>

```

With just one gate in the pipeline `q`, the glue `p` is unused:

```
~zod:dojo> (;~(cmp trip) 'a')
"a"
```

But for multiple gates, we need it to connect the pipeline:

```
~zod:dojo> (;~(cmp trip |=(a=@ ~[a a])) 'a')
"aa"
~zod:dojo> (;~(cmp trip |=(a=@ ~[a a])) '')
""
```

A more complicated example:

```
~zod:dojo> (;~(cmp trip ;~(cmp |=(a=@ ~[a a]) |=(a=@ <(dec a)>))) 'b')
"97b"
~zod:dojo> (;~(cmp trip |=(a=@ ~[a a]) |=(a=@ <(dec a)>)) 'b')
"97b"
~zod:dojo> (;~(cmp trip |=(a=@ ~[a a]) |=(a=@ <(dec a)>)) '')
""
~zod:dojo> (;~(cmp trip |=(a=@ ~[a a]) |=(a=@ <(dec a)>)) 'a')
"96a"
~zod:dojo> (;~(cmp trip |=(a=@ ~[a a]) |=(a=@ <(dec a)>)) 'acd')
"96acd"
```

### `;*` "mictar"

make a list of XML nodes from complex Hoon expression (Sail)

##### Produces

A `marl`, i.e., a list of `manx`.  A `manx` is a noun that represents a single XML node.

##### Syntax

**1-fixed**

```hoon
;*  p=hoon
```

`p` is a Hoon expression that produces a `marl`.

##### Discussion

`;*` is a Sail rune.  Sail is a part of Hoon used for creating and operating on nouns that represent XML nodes.  With the appropriate rendering pipeline, a Sail document can be used to generate a static website.

If you need a complex Hoon expression to produce a `marl`, use the `;*` rune.  Often this rune is used with an expression, `p`, that includes one or more `;=` subexpressions.

(See also [`;=`](#mictis).)

##### Examples

```
> ;*  ?:  (gth 3 2)
        ;=  ;p: This is node 1 of 'yes'.
            ;p: This is node 2 of 'yes'.
        ==
      ;=  ;p: This is node 1 of 'no'.
          ;p: This is node 2 of 'no'.
      ==
[ [[%p ~] [[%$ [%$ "This is node 1 of 'yes'."] ~] ~] ~]
  [[[%p ~] [[%$ [%$ "This is node 2 of 'yes'."] ~] ~] ~] ~]
]

> ;*  ?:  (gth 2 3)
          ;=  ;p: This is node 1 of 'yes'.
              ;p: This is node 2 of 'yes'.
          ==
        ;=  ;p: This is node 1 of 'no'.
            ;p: This is node 2 of 'no'.
        ==
[ [[%p ~] [[%$ [%$ "This is node 1 of 'no'."] ~] ~] ~]
  [[[%p ~] [[%$ [%$ "This is node 2 of 'no'."] ~] ~] ~] ~]
]
```

### `;=` "mictis"

make a list of XML nodes (Sail)

##### Produces

A `marl`, i.e., a list of `manx`.  A `manx` is a noun that represents a single XML node.

##### Syntax

**running**

```hoon
;=  p=hoon  q=hoon  ...  z=hoon  ==
```

`p`-`z` are Hoon expressions, each of which poduces a `manx`.

##### Discussion

`;=` is a Sail rune.  Sail is a part of Hoon used for creating and operating on nouns that represent XML nodes.  With the appropriate rendering pipeline, a Sail document can be used to generate a static website.

In Sail a single XML node is represented by a `manx`.  A single `<p>` node `manx` can be produced in the following way:

```
> ;p: This will be rendered as an XML node.
[[%p ~] [[%$ [%$ "This will be rendered as an XML node."] ~] ~] ~]
```

Sometimes what is needed is a `marl`, i.e., a list of `manx`.  To convert a series of `manx` nodes to a `marl`, use the `;=` rune.

(See also [`;*`](#mictar).)

##### Examples

```
> ;=  ;p: This is the first node.
      ;p: This is the second.
      ;p: Here is the last one.
  ==
[ [[%p ~] [[%$ [%$ "This is the first node."] ~] ~] ~]
  [[%p ~] [[%$ [%$ "This is the second."] ~] ~] ~]
  [[%p ~] [[%$ [%$ "Here is the last one."] ~] ~] ~]
  ~
]
```
