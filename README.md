# Gamma

[![Build Status](https://travis-ci.org/parsonsmatt/gamma.svg?branch=master)](https://travis-ci.org/parsonsmatt/gamma)

A toy programming language that seeks to reify gamma.

## Concept

Look at this Haskell function declaration:

```haskell
idInt :: Int -> Int
idInt = \x -> x
```

What does this mean?
It means that we are declaring a term `idInt` into the module's scope.
What is the scope?
Well, in a sense, it is the set of terms that we can refer to without having a
compilation error.
When we're defining `idInt`, we declare that it is equal to a lambda.
This lambda introduces another term, `x`, which is only in scope in the body of
the lambda.

Does `idInt` depend on anything?
Not at the term level!
However, the type signature *does* have a *free* term: `Int`.
`Int` must be defined somewhere else: it must already be in scope.

Free terms represent a dependency of the function on the ambient environment.
The compiler currently checks that all free terms are accounted for.
This prevents compilation problems, but it discards useful information:
specifically, the expected context of a function.

What is another option? Let us consider a new bit of notation, `Gamma`, that
declares the free variables that a function expects.

```haskell
idInt :: Int in Gamma |- Int -> Int
idInt = \x -> x
```

Fortunately, this is easy to infer.
We should also want to be able to specify terms:

```haskell
logInput :: log in Gamma |- String -> IO String
logInput = do
    input <- getLine
    log input
    pure input
```

`log`, in this case, is inferred to have the type `String -> IO ()`.
As above, this term is easy to infer.
What useful thing might we want to state with this?

Suppose a module:

```haskell
module LogInput where

import Prelude (String, IO, getLine, pure)

logInput :: log in Gamma |- String -> IO String
logInput = do
    input <- getLine
    log input
    pure input
```

What *is* this module?
It doesn't compile.
It is missing a term: `log`.
The compiler doesn't know what to do with it, so it spits out an error about a missing variable `log`.

What *could* it be?
Consider a lambda term `\x : Int . f x : Char`.
`f` is a free variable.
If `f` isn't in scope, then we can *parameterize* this lambda on `f`, resulting in:
`\f : Int -> Char . \x : Int . f x`.
So a module (or function?) with Gamma variables becomes a *lambda* that, when provided a term of the correct type (or a type of the correct kind?), returns a module.
