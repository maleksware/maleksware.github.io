---
layout: post
title: How to learn Haskell in COMP1100
---

I have completed COMP1130 in S1 2025 with a relatively high mark. I've heard a lot of stuff (from fellow coursemates and tutors) while doing the course that made me write this post.

If you're worried about your level of prep for the course or just want to have a different perspective on this (undoubtedly exotic) course, read on.

# How to approach COMP1100/COMP1130 as a first year student

You might never have heard of Haskell before. That is totally fine. (This is what they'll tell you on the first lecture). The main reason Haskell is the language taught
in this course is to level the playing field and not let anyone with 2-3 years of Python experience alone dominate the exams and assignments.

An unpopular opinion: if you've never done programming before, Haskell may be easier for you to learn than if you're proficient in, say, Java or C++.

Also, recall that the name of this course is not "Haskell 101", it's "Programming as Problem Solving". It's not to teach you the language, it's to mess with your brain such that by the
end of the course you have a better thinking and problem-solving setup.

There are 3 sections to this post - something to read before you start the course (general points), technical details (read in the middle of the course) and comments on type derivations (for COMP1130 students).

## General remarks

Programming languages are generally either _imperative_ or _functional_. Most modern languages have features from both of those groups. Imperative languages are those that have specific commands that you can run in order, with the goal of achieving your task. For example, the following Python program gives a sense of what this means:

```python
a = 15  # declaring & initialising variables
b = int(input("Enter the other number"))  # reading user input
c = 0
if b > 10:  # conditional
    c = a * b - 12  # change the value of c
else:
    c = 5
print("Result is " + str(c))
```

This program reads a number from the user and does some calculations. All instructions are executed in order from top to bottom. Note that we explicitly say what we want to be done.

Functional languages, like Haskell, follow a different pattern: everything in Haskell is an expression, e.g., a construction that has a concrete value that can be computed. For example, `13 + 3` is an expression, whereas in most languages something like `print("hello")` is not (what is a reasonable value of printing stuff?) Haskell is built in a way such that everything you write is an expression, even `if`s. As such, Haskell has no "instructions" (known as _statements_). It has functions that you can apply to things to get the result you want. Moreover, Haskell's functions are _pure_, meaning that running them any number of times on the same data doesn't change the result _and_ the state of the outside world. Think of them as mathematical functions - do you know a mathematical function that prints stuff?

This has an important consequence. By the end of COMP1100/1130, you will not know how to print anything to the terminal. It may seem like the language Haskell is therefore completely useless, but that's not true - you just need more advanced features outside the scope of the course to change the world around you with Haskell code.

From here, a number of general points follow:

1. **Do not** try to make your common constructions from Python/C/... work in Haskell. This language does not, and should not, have loops. If you consider how recursion (more about it later) helps you
solve the problem, you'll realise you never really needed loops to solve problems from this course. Holding onto imperative languages' constructions will not give you the
insights this course is designed to provide.

2. Haskell has really simple syntax. Essentially (in this course anyway), it's built around 5 things: function application, pattern matching, currying (aka partial application), recursion and polymorphism. Understanding how each of those things works (**by going to labs and working on lectures**) will give you all you need to know about Haskell.

3. Really **do** listen in the first lectures of the course that teach you about functions and sets. Not understanding that functions are mappings between sets will make your experience painful. After all, Haskell's type system and runtime semantics (how the program is executed) are both based on sets and functions between them.

4. **Go to labs. Practise daily. Work on lectures.** By "working on lectures" I mean attending them in person, when possible, and listening actively. Ask questions! If your convener remembers you after the course, you can consider yourself a lecture worker.

5. **Solve challenging problems.** Try the optional ones, even if you're in COMP1100. By not doing them, you (excluding situations with really dense timetables, etc) essentially write on your forehead: "I don't want to know how to do the cool stuff". I know I will sound really, I don't know, *not-down-to-earth*, but giving up on trying to actually get knowledge in your first computing course in your first year is not the best thing.

    One of the best ways to get better in the course, get respect of your coursemates and boost social skills is **helping people on the forum**. That is a really underrated thing.

## Technical points

1. The hardest theoretical thing is recursion (functions calling themselves). It is cornerstone to every single problem that you will have to solve to get a pass. There are millions of ways people explain it online and at ANU, and the latter are often really great. I think of recursion as a promise (not to be confused with `Promise` in JS/TS/...). When implementing a recursive function, I think in the following way:

    > Here is a problem of some size to solve. I split this problem into a smaller similar problem and a very small chunk of data. If someone from the sky sends me the solution to that similar problem, I can combine that solution with the chunk of data and solve the original problem.

    > But my problems are very similar. I'm implementing a function, say, `f`, that solves this type of problems. Why not use this function I promised to myself to implement to send me the solution from the sky (recursive case)?

    > If I keep getting solutions from the sky (recursive call), I will never finish processing the problem (infinite recursion). I have to do a manual consideration of a case that is simple enough for me to solve myself (base case).

    By thinking like this, I can implement a function that computes the sum of the numbers in the list:

    ```haskell
    sumList :: [Int] -> Int
    sumList nums = case nums of
        [] -> 0  -- base case - sum of numbers in an empty list is 0
        x:xs -> x + sumList xs
        -- recursive case: smaller problem is finding the sum of everything
        -- except the first element of a nonempty list.
        -- If I get the solution to that from the sky,
        -- I can add my first element to that solution and get the
        -- sum of the elements of the initial list.
        -- But my function has been promised to find the sum of the elements
        -- in the list anyway, so why not use that?
    ```

2. A note on syntax: it is not commonly explained that way, so I'll explain it here. You may remember that the syntax for guards is really confusing sometimes. Guards (`|`-expressions), as taught in COMP1100, can be used in function definitions and `case ... of` expressions. That is correct, but then it's hard to remember where `=` and `->` go in those expressions (especially when you have nested case-ofs).

    I have seen a great comment that "guards are restrictions on pattern matching". Function definitions are pattern matching: the following snippets are equivalent.

    ```haskell
    f1 :: Maybe Int -> Maybe Int
    f1 (Just x) = Just (x * 2)
    f1 Nothing  = Nothing
    ```

    ```haskell
    f2 :: Maybe Int -> Maybe Int
    f2 mx = case mx of
        Just x  -> Just (x * 2)
        Nothing -> Nothing
    ```

    *The former way is not recommended because if you forget a case, your function will be partial
    (undefined for some inputs) and that's bad. You can't forget a case in a `case ... of` because the code will not compile.*

    **Back to the point**: Guards can be placed after any pattern matching. Be careful to consider all cases - the compiler cannot verify that you've covered all possibilities in guards. (Use `otherwise`.)

    Consider the following: a function that tries to divide a `Maybe Int` value by 2. If the number inside of the `Maybe` is odd, the function returns `Nothing`.

    ```haskell
    safeDivide :: Maybe Int -> Maybe Int
    safeDivide mx = case mx of
        Just x | x `mod` 2 == 0 -> Just (x `div` 2)
               | otherwise  -> Nothing
        _ -> Nothing
    ```
    
    See how the guards are placed right after we matched on `Just x`? And the arrow after the guard expression is just the arrow from the pattern match. It works exactly the same way with function definitions, too:

    ```haskell
    safeDivide :: Maybe Int -> Maybe Int
    safeDivide (Just x) | x `mod` 2 == 0 = Just (x `div` 2)
                        | otherwise  = Nothing
    safeDivide _ = Nothing
    ```
    Actually, what do you think `otherwise` does? It's not a crucial language construction: if you replace it by `True` everywhere, nothing will change (why?) This suggests that `otherwise` is just an _alias_ (alternative equivalent version) for `True`: if you type `:t otherwise` or even just `otherwise` into GHCi, you'll see that it is precisely `True`. (Nothing prevents you from using `otherwise` instead of `True` elsewhere in your code, but this is terrible style and the marker will likely raise their eyebrow and give you 0 marks for that.)

3. Embrace higher order functions. This will be the main piece of advice you'll get when preparing for assessments, but for real, you should know how to implement `map`, `foldr`, `foldl` and sometimes `filter` for every data structure you see in the course (`BinaryTree` and `RoseTree` being the main examples). You should really understand how associativity affects the results of `fold`, and how that is reflected in implementations.

4. Use Hoogle. This actually deserves a separate subsection:

### Using Hoogle

[Hoogle](https://hoogle.haskell.org) is your best friend for the entirety of this course. You can look up:
- Functions by name
- Functions by type signature (search for `(a -> b) -> [a] -> [b]`, it'll give you `map` for lists)
- Typeclasses
- Specific instances of typeclasses (is `Integer` an instance of `Ord`?)
- and lots more.

Examples are sometimes complicated but should give you an idea of how simple things like `map` work.

A thing a lot of students overlook is the ability to view the source code for a function: if you're looking at a page for a function in Hoogle, to the very right of the type signature there is a `Source` link
that brings you directly to the definition of the function in `Prelude`. This is where you'd typically find the finest Haskell code, even for simple functions.

## Simply typed lambda calculus

This section is for miscellaneous advice related to type derivations. TBC

## Other things

Stuck? Get unstuck by using the forum, Hoogle, labs and fellow coursemates.

The course channel in the CSSA Discord has a bunch of experienced people willing to help. It will be active during Semester 1 - ask questions, help others!

**Good luck!**
