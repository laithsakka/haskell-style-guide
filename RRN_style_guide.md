Haskell Style Guide, diff from Tibbe's
======================================

This document captures an alternative Haskell style guide that extends
[Johan Tibbell's](https://github.com/tibbe/haskell-style-guide/blob/master/haskell-style.md)
 with the following tweaks.

 * Relax width restriction to 100 columns.
 * Allow 2-space everewhere 4-space indents are specified.

Additionally, this style guide specifies additional rules below that
extend, but do not conflict with, the original guide.


Choice of case versus multiple function equations
-------------------------------------------------

This choice is based on the length of the overall function, number of
cases, and the line counts of the individual cases.

### Short functions:

For short functions with few cases, using multiple equations is fine:

```Haskell
f :: Maybe Int -> Int
f Nothing  = 0
f (Just n) = n
```

In this example, the function definition is only three lines long.
Going to a case expression would add 1-2 extra lines and would also
add a variable binding which is just clutter.  No need to name, or
think about, that binding.

### Medium-length functions

As the number of cases, and the length of the function name
itself grows, this style becomes unrecommended:

```Haskell
freeVars :: Exp -> Set Var
freeVars (VarE v)        = ... short expr ...
freeVars (LitE _)        = ... short expr ...
freeVars (LitSymE _)     = ... short expr ...
freeVars (AppE _v _ ls)  = ... short expr ...
freeVars (PrimAppE _ ls) = ... short expr ...
...
```

Here, the repeated occurrence of the identifier `freeVars` is just
clutter.  More edges for the eye to parse needlessly.  Further,
indentation as a signal has been taken away.  In this example
indentantion communicates nothing.

The point here is more about reading than writing.  While the above is
*tedious* to write by hand, that of course may be automated by emacs
modes, IDEs, case spliting etc.  Nevertheless, it remains that with
today's tools the above style is harder to *refactor* than a case
expression.  If we used a single case expression instead of multiple
equations, we could:

 * copy/paste that case expression
 * indent that case expresssion more deeply to wrap a `let` around it
 * wrap a `where` clause around it
 * refer to the original variable binding without using an `v@` pattern.

For all these reasons we would prefer the below style for this
medium-length function, either with or without a hanging `case` expr:

```Haskell
freeVars :: Exp -> Set Var
freeVars e0 = case e0 of
  VarE v        -> ... short expr ...
  LitE _        -> ... short expr ...
  LitSymE _     -> ... short expr ...
  AppE _v _ ls  -> ... short expr ...
  PrimAppE _ ls -> ... short expr ...
  ...
```

Here we also save an extra level of parentheses.

### Long functions

For very long functions, the calculation changes again:

```Haskell
f :: Foo -> Int
f (A _) = ... > 1pg of code...

f (B _ _) = ... > 1pg of code...
```

In this case, the multiple-equations style is preferred, because it
helps reorient the reader when the first `f`-definition line is
already off the screen.

Of course, if a function is very long, that is *also* a good reason to
perform more refactoring, introduce helper functions, and reduces its
size.


Alignment of arrows and equals
------------------------------

In the above examples, the cost of horizontally aligning the `=`
symbols and the `->` symbols was small, and greatly increases
readability.  

In some cases, patterns are of highly variable lengths, and it is
acceptable to "gracefully degrade" into blocks of likewise-aligned
cases:

```
f :: Foo -> Int
f x =
  case x of
    (A _)   -> _
    (B _ _) -> _
    (C _)   -> _
    (VeryLongNameHere _ _ _) -> _
    (PrettyLong _ _ _)       -> _
    ...
```

This retains some of the visual benefits of fully aligning the arrows,
without requiring excessive numbers of spaces that create awkward
looking cases like:

```
    (A _)                    -> _expr
```

