This is a small reference implementation of a subtyping type inferencer.

Example of input:

    fibonacci(n)
        if gt(n, 2)
        then add(fibonacci(sub(n,1)), fibonacci(sub(n,2)))
        else 1

And output for that fibonacci:

    add :: (add)
    sub :: (sub)
    call :: (call)
    gt :: (gt)
    fibonacci :: ((gt & add & a0) -> (int))
      gt : (((a0), (int)) -> (bool))
      add : (((a0), (int)) -> (gt & add & a0))
      add : (((a0), (int)) -> (gt & add & a0))

The work in this directory is based on Stephen Dolan's
[MLsub](https://www.cl.cam.ac.uk/~sd601/mlsub/). The goal of
this project was to extend his work into a verifiable programming
language that is flexible enough to dual as a computer algebra system.

I succeeded.

The main discovery in this project was this clause:

    B is a set of type constructors.
    exist! y in B. forall x in B. x --> y

The above clause can be used for implementing extensible type promotion. 

An example: operator `a + b`.

 1. Find type for `a` and `b`, form `B` out of them.
 2. Apply the condition. Either the set has only one item.
    Or then the `a --> b` or `b --> a`. If both occur at same time,
    we cannot do unique coercion.
 3. `y` is the unique type that fullfilled the condition.
 4. Convert `a` and `b` to `y`. Pick implementation for `+`
    from `y`. Evaluate the result.

The language used in this repository is the previous version of
[Lever](https://github.com/cheery/lever). It resembles Python, although
current Lever it is probably different when you get to read this.

## Coherent coercions

Nicolas Doye presented an algorithm for automatically
creating coercions between types in his paper
["Order Sorted Computer Algebra and Coercions"][doye]. His
system required:

 1. coherent coercions between base types
 2. direct embedding coercions
 3. structural coercions
 4. identity function on ground types as coercions
 5. composition of coercions
 6. homomorphisms between types

The resulting system would form coherent set of coercions,
meaning that different ways to coerce from any one type to
an another type produce the same result.

 [doye]: http://axiom-wiki.newsynthesis.org/public/refs/doye-aldor-phd.pdf

Subtyping as it shows up in Dolan's MLSub doesn't perfectly
convey the idea of coercions. To see why, consider the expression:

    (a + 3).x

Assume that `(+)` is typed as `(a, a) -> a`, we would end up with
the following constraints.

    int <: getattr_x(x)
    a   <: getattr_x(x)

`int` type has no way to provide `.x` -attribute, but the expression
could become valid if `a` became a vector, because we could then
lift `int` inside the vector. In this case though, the `(a, a) -> a`
type signature for `(+)` does not hold.

To make this work, the operators must be implemented a bit like typeclasses,
and the types must be filled in whenever there is an operator which no longer
has external inputs that would change the value.
