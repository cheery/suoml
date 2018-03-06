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

 [doye]: (http://axiom-wiki.newsynthesis.org/public/refs/doye-aldor-phd.pdf).

Dolan's work presents an ideal base for typing this kind of
a system. The algorithm presented in the paper supports
structural coercions as it stands. We end up needing to come
up a way to represent:

 1. direct embedding coercions
 2. homomorphisms between types

I think these properties can be achieved by representing the
types supporting embedding as "modifier" types that are
slapped over base coercions. The type signature in MLsub
system for such object would be of the form:

    embedding_type | base_type

In comparison to the non-embedding types that are of the
form:

    non_embedding_type(base_type)

But this presents a problem. Lets consider simple embedding
of complex values.

It would be trivial to implement an add function in MLsub
that is able to carry out the following operation, and
return the associated type.

    (1 + 3j) : (complex | float)

The problem is, how do we exctract `real` and `imag` from
this? We need the corresponding functions.

    real :: complex -> ?
    imag :: complex -> ?

To achieve this we need to introduce "cancellation".

    real :: complex & (a !! complex) -> a
    imag :: complex & (a !! complex) -> a

With some additional ruling, the embedding coercions could
stack and form some difficult types, given that those types
form homomorphisms or otherwise provide themselves a fixed
order. The following type could resolve for the operation
taking the real component from a matrix containing complex
values.

    (float & matrix(2,2) & complex) -> (float | matrix(2,2))

The goal of this project is to find out how to extend the
algorithm to support this cancellation operator in the type
inference, and then verify it forms a system that works as
expected.
