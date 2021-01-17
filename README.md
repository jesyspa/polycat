# polycat

A functional programming language that is automatically category-polymorphic.  This work is in the very early stages;
there is currently no implementation, nor a comprehensive understanding of whether this would work at all.

## Idea

A function in a programming language like Haskell can be seen as a lambda term (perhaps using some additional features
like inductive types), which in turn can be seen as a construction in some Cartesian closed category.  In Haskell, this
category is colloquially known as Hask.

However, placing definitions to a single category is overly restrictive.  For example, the definition
```haskell
id :: a -> a
id x = x
```
is in fact valid in every category.  For a more complicated example,
```haskell
(+) :: N x N -> N
zero + n = n
succ k + n = succ (k + n)
```
requires only product type and a natural numbers object.  By restricting to a single category, we cannot use these same
definitions for other categories such as the Kleisli category or the product category.

In Polycat, these definitions are by default polymorphic in what category they should be interpreted in, allowing code
reuse across categories.

## Examples

Let us imagine what Polycat could one day look like.  Suppose we would like to compute the squared norm of a 2-vector in
Polycat.  This can be done as follows.  Note that we use a : A to indicate a is of type A.

```agda
square : N -> N
square x = x * x

sqnorm : N x N -> N
sqnorm v = (+) ((square from C^2) v)
```

Here, `square in C^2` indicates that we reinterpret square as taken from the category `C^2`, where `C` is the category
we are currently working in.

In more advanced cases it may be necessary to specify extra requirements on a category.

```agda
module _ where
  requires get : N
  requires set : N -> ()

  -- A stateful computation that uses get and set to track the state.
  foo : N -> N

bar : N -> N
bar n = fst ((foo from State N C) foo n zero)
```

Here, `State N C` is the Kleisli category for the State monad.  The morphism `foo : N -> N` in the Kleisli category is
interpreted as a morphism `N -> N -> (N, N)` in category `C`.  Note that `State`, being a transformation of categories,
is automatically a monad transformer.

## Goals for minimal example

Initially, I am interested in supporting some base category that is Cartesian and has a natural numbers object, and some
categories formed from this by finite products.  The relations between the categories do not yet need to be part of the
language.  This should suffice to implement the square norm example above.

## Unresolved Questions

Once again, this is a very early draft.

### How are categories defined and related?

Ideally, we would like the user to be able to specify a completely new category and construct all the structure (e.g.
products, exponentials) inside it.  However, it is not clear how to allow the user to build something outside the
category already provided to them.

### How do we deal with currying?

If when the user defines a function `f : A -> B` this is given by some morphism, then the introduction of higher-order
functions leads to a decrease in uniformity.  Namely, a definition of a function `g : A -> (B -> C)` now uses `->` in
two senses: the left `->` indicates a morphism, while the right indicates the construction of an exponential object.

### How do we deal with referential transparency?

If we allow the user to work in the Kleisli category as in the stateful example above, we may obtain expressions like
`rand + rand : Rand Int` where two random numbers are generated.  However, it is unclear whether moving the duplicate
occurence into a let binding would ensure that only one random number is generated.
