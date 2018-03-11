The programming language used to implement this language is
[Lever](https://github.com/cheery/lever). The current plan is to
resume and redesign that language once I have figured out
the details in this small language.

The work in this directory is based on Stephen Dolan's
[MLsub](https://www.cl.cam.ac.uk/~sd601/mlsub/). The goal is
to extend his work into a verifiable programming language
that is flexible enough to dual as a computer algebra system.

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

Dolan's work would appear to be an ideal base for typing this kind of
a system, yet the main problem of representing coercions remain.

The reason why problem persist is that subtyping doesn't
perfectly convey the idea of coercions. To see why, consider
the idea below:

    (a + 3).x
    ----------------------
    int <: getattr("x", x)
    a   <: getattr("x", x)

If we went by representing coercions as subtyping
expressions, we would face the puzzle as above.

`int` type itself has no way to provide an `.x` -attribute,
but it could provide that given it gets promoted into a
vector.

    (3).x
    ----------------------
    int <: getattr("x", x)

If we take a different case where the `int` type ends up
being directly passed to retrieve `.x`, we can see that we
have a problem here.

The problem is that we cannot assume that there exist only
one type that `int` can be promoted into, that has `.x`.

We may also decide that the coercion must be speudo-implicit
and tied to the arithmetic operations. In this case the type
of addition would be:

    (+) :: g -> g -> b, where g = a & coercible(b, a)

Now the above expression `(3).x` would correctly present it
has a type inference problem. Whereas the expression below
would type:

    (a + 3).x
    ------------------------
    b <: getattr("x", x)
    a <: coercible(b, a|int)

We could still get into a situation where the program does
not provide correct type:

    (15 + 3).x
    -----------------------
    b <: getattr("x", x)

Although now this happens because the type inference no
longer concerns over the problems of coercibility.

We may face an another problem if we wanted the types to
extend over dependently typed concepts such as matrices. For
matrices the types of addition and multiplication should be:

    (+) :: matrix(a,b) -> matrix(a,b) -> matrix(a,b)
    (*) :: matrix(a,b) -> matrix(b,c) -> matrix(a,c)

The matrix addition would appear to satisfy the rules we
have laid. Partially because it's element-wise addition. The
multiplication in other hand presents a trinary relation for
us to solve. Eliminating the details of matrices leaves us
with:

    (*) :: a -> b -> c

We might be able to abstract this out from the expression.

    (*) :: a&g -> b&g -> c where g = rel("*",+a,+b,-c)

This treatment would hoist the unsolvable trinary relations
into the arguments of the function.

Further experimentation is required to see whether this is a
feasible approach to solve this problem.
