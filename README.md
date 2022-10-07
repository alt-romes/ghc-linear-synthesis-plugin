## GHC Linear Synthesis Plugin

Synthesis is the automatic generation of a program given a specification.

This GHC-plugin is a synthesizer that takes a (linear) type as a specification
and synthesizes a program that inhabits that type.

It makes use of the (more expressive) linear types to heavily constrain the
valid program search space -- which leads to more accurate results, faster.

The synthesis is constructed, through the Curry-Howard isomorphism, as bottom-up
proof search in linear logic (if we find a proof for a type, we have
synthesized a program that inhabits that type).

We can then leverage the existing literature on linear logic and proof search,
and employ two proven techniques that remove the don't-care non-determinism
from proof search in linear logic: resource-management[1] and focusing[2]

The result is a fast and correct-by-construction synthesizer whose specification
is a linear type -- which is expressive enough on its own to lead to the
synthesis interesting programs.

[1] - Cervesato, I., Hodas, J.S., Pfenning, F.: Efficient resource management for linear logic proof search. Theor. Comput. Sci. 232(1-2), 133–163 (2000). https://doi.org/10.1016/S0304-3975(99)00173-5

[2] - Andreoli, J.M.: Logic Programming with Focusing Proofs in Linear Logic. Journal of Logic and Computation 2(3), 297–347 (06 1992). https://doi.org/10.1093/logcom/2.3.297

## Expressiveness

A simple example of the expressiveness of linear types is the `map` function.

If we were to ask a synthesizer which doesn't account for linearity to
synthesize a function of type
```hs
map :: (a -> b) -> [a] -> [b]
```
The synthesizer might output
```hs
map _ _ = []
``` 
Which does inhabit the prompt type. Real synthesizers might overcome this
difficulty of understanding intent by requiring a more fine-grained
specification that complements the simple type (e.g. type + examples, refinement
types, natural language description, ...)

In turn, by using linearity we can fully express our intent with map: a function
which consumes all `a` elements of the list exactly once and maps them to `b`
(the only way it knows how). With the prompt
```hs
map :: Ur (a %1 -> b) %1 -> [a] %1 -> [b]
map = _
```
This synthesizer will generate (well, something equivalent to, but definitely
not as pretty (yet))
```hs
map (Ur f) = \case
    [] -> []
    ((:) x xs)
        -> (:) (f x) (map (Ur f) xs)
```
Which is indeed the map function we want to synthesize. Because we must consume
the list exactly once, we are forced to use the function on each element because
it's the only way of making `b`s.


## Using


## Building

Compilation is possible with ghc-9.0 and ghc-9.2.
We don't yet support ghc-9.4 because `ghc-source-gen` hasn't caught up with it.


