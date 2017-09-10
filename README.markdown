# repl-check: Test interactive Haskell examples

[![Build Status](https://travis-ci.org/chris-martin/repl-check.svg)](http://travis-ci.org/chris-martin/repl-check)

`repl-check` is a small program that checks examples in a simulated GHCi session.

`repl-check` is a fork of `doctest` which, rather than looking for tests in Haskell comments, reads them from a separate file.

## Usage

Below is a small example file named `fib.txt`. The file contains some examples of interaction; the examples exist in part to demonstrate how some module is supposed to be used, and in part as tests.

```
'fib' compute Fibonacci numbers.

Examples:

λ> fib 10
55

λ> fib 5
5
```

A line starting with `λ>` denotes an _expression_. All consecutive nonempty lines following an expression denote the _result_ of that expression. The result is defined by what the REPL prints to `stdout` and `stderr` when evaluating that expression.

You may check whether the given expressions in `fib.txt` actually produce the output that follows them by typing:

    repl-check fib.txt

### Multi-line input

GHCi supports commands which span multiple lines, and the same syntax works for `repl-check`:

```
λ> :{
  let
    x = 1
    y = 2
  in
    x + y + 3
:}
6
```

### Multi-line output

If there are no blank lines in the output, multiple lines are handled automatically.

```
λ> putStr "Hello\nWorld!"
Hello
World!
```

If however the output contains blank lines, they must be noted explicitly with `<BLANKLINE>`. For example,

```
λ> import Data.List (intercalate)

'doubleSpace' double-spaces a paragraph.

Examples:

λ> let s1 = "\"Every one of whom?\""
λ> let s2 = "\"Every one of whom do you think?\""
λ> let s3 = "\"I haven't any idea.\""
λ> let paragraph = unlines [s1,s2,s3]

λ> putStrLn $ doubleSpace paragraph
"Every one of whom?"
<BLANKLINE>
"Every one of whom do you think?"
<BLANKLINE>
"I haven't any idea."
```

### Matching arbitrary output

Any lines containing only three dots (`...`) will match one or more lines with arbitrary content. For instance,

```
λ> putStrLn "foo\nbar\nbaz"
foo
...
baz
```

If a line contains three dots and additional content, the three dots will match anything *within that line*:

```
λ> putStrLn "foo bar baz"
foo ... baz
```

### Cabal integration

`repl-check` provides both an executable and a library. The library exposes a function `replTest` of type:

```haskell
replCheck :: [String] -> IO ()
```

`repl-check`'s own `main` is simply:

```haskell
main = getArgs >>= replCheck
```

Consequently, it is possible to create a custom executable for a project, by passing all command-line arguments that are required for that project to `repl-check`.  A simple example looks like this:

```haskell
-- file repl-check.hs
import Test.ReplCheck
main = replCheck ["test/repl-fib.txt"]
```

And a corresponding Cabal test suite section like this:

    test-suite repl-check
      type:          exitcode-stdio-1.0
      ghc-options:   -threaded
      main-is:       repl-check.hs
      build-depends: base, repl-check
