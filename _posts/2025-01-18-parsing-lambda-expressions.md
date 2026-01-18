---
layout: post
title: Parsing λ-expressions with Haskell
---

The commonly known textbook grammar for lambda expressions,

```
M ::= x | λx.M | M M
```

or its even more widely known form

```
M ::= x | λx.M | (M M)
```

is, surprisingly, absolutely not suitable for parsing (it is not a formal lexical grammar). There are two main downsides:

1. It is ambiguous. Both of the above "grammars" would allow parsing something like `λx.y x` (or, in the case of the second grammar, `(λx.y x)`) as either
`[λx. y] [x]` or `[λx. [y x]]`. As abstractions bind all the way to the right, the latter reading is correct, but a naive parser will output the former one
(if it even succeeds, as there is no guarantee that the remaining ' x' part will be correctly interpreted as an application if the parser has already started
parsing the abstraction `λx.y` as a standalone term). 

2. Parentheses are unloved. Something technically valid like `(λx.x)` or `(x y) z` is not a valid sentence in any of the above grammars. Alas, computer scientists understand what those expressions mean, as parentheses operate on the level above the defined grammar. Ideally, we want the parser to be robust and parse expressions that humans would come up with.

## Fixing the grammar

First, let's allow parentheses. They are always handy, and they are crucial tokens in some grammars where left recursion shows its ugly head.

```
M ::= x | λx.M | M M | (M)
```

Much better. We can avoid using whitespace completely by forcing application to use parentheses around each term (`(x)(y)` instead of `x y`), but let's leave it as a conventional separator.

Observe, however, that even though the second issue is no longer present, the first one is still to be fixed: applications can be parsed either way, and if we parse them naively with a left recursive parser, we'll get associativity wrong (application is left-associative).

## Setting up the parser

The code below is fairly standard. We're implementing a small parser combinator utility (an explanation can be found [here](https://serokell.io/blog/parser-combinators-in-haskell)). For simplicity, we will use `Maybe` instead of `Either`.

```haskell
module Lambda where

import Data.Char
import Control.Applicative

newtype Parser a = Parser {
  runParser :: String -> Maybe (a, String)
}

instance Functor Parser where
  fmap f (Parser p) = Parser $ \inp -> do
    (r, rest) <- p inp
    return (f r, rest)

instance Applicative Parser where
  pure c = Parser $ \inp -> Just (c, inp)
  Parser f <*> Parser p = Parser $ \inp -> do
    (fn, rest) <- f inp
    (p', rest') <- p rest
    return (fn p', rest')

instance Alternative Parser where
  empty = Parser (const Nothing)
  Parser u <|> Parser v = Parser $ \inp -> u inp <|> v inp
```

We can leverage the fact that `Maybe` is, in fact, satisfying all three typeclasses above, and make the code quite concise. We can implement the `Monad` for the parser as well, but it's not needed for a simple parser.

Let's introduce some useful building blocks for the parser:

```haskell
satisfy :: (Char -> Bool) -> Parser Char
satisfy pred = Parser $ \inp ->
  case inp of
    x:rest | pred x-> Just (x, rest)
           | otherwise -> Nothing
    _      -> Nothing

charP :: Char -> Parser Char
charP c = satisfy (==c)

stringP :: String -> Parser String
stringP = traverse (\c -> satisfy (==c))

sepBySome :: Parser a -> Parser b -> Parser [a]
sepBySome p sp = (:) <$> p <*> many (sp *> p)

sepBy :: Parser a -> Parser b -> Parser [a]
sepBy p sp = sepBySome p sp <|> pure []

ws :: Parser String
ws = many (satisfy isSpace)
```

(Again, the explanations can either be inferred from definitions or looked up. The naming used here is either conventional or used by [Tsoding](https://www.youtube.com/watch?v=N9RUqGYuGfw) in his JSON parsing video.)

Now, let's introduce a result type for the parsed lambda expressions. There are many ways one can do this, and I wouldn't say that this is the best one. We will have one constructor for each element in the original grammar:

```haskell
type Variable = String

data LExpr = Var Variable
           | App LExpr LExpr
           | Abs Variable LExpr
  deriving (Eq, Show)
```

Individual mentions of variables are wrapped in constructors (to be resolved), and variables bound by the closest `λ` are just strings. It's not hard to support syntactic sugar for currying in expressions like `λx y z. z (y x)`, but we'll omit it for simplicity.

Now, let's see how we can parse the actual expressions. We could do it naively by following the grammar explicitly:

```haskell
expression :: Parser LExpr
expression = variable <|> abstraction <|> application <|> grouping

variable :: Parser LExpr
variable = Var <$> literal

-- A nonempty sequence of valid identifier characters. Numbers are allowed.
literal :: Parser Variable
literal = Parser $ \inp ->
  let (xs, rest) = span (\c -> isDigit c || isAlpha c) inp
  in (if null xs then Nothing else Just (xs, rest))

abstraction :: Parser LExpr
abstraction = (\_ v _ e -> Abs v e) <$> charP lambda <* ws
                                    <*> literal <* ws
                                    <*> charP '.' <* ws
                                    <*> expression
lambda :: Char
lambda = 'λ'

application :: Parser LExpr
application = (\f _ a -> App f a) <$> expression <*> some ws <*> expression

grouping :: Parser LExpr
grouping = charP '(' *> expression <* charP ')'
```

It would've been great if this was correct, but the implementation above has two problems: it gets associativity of application wrong (`f x y z` would parse as `f (x (y z))` not as `((f x) y) z`) and it gets stuck in left recursion on parsing any applications. By definition of `Alternative`, if parsing an individual variable fails, and the expression is not an abstraction, then it must be an application. An application, by its definition, immediately tries to parse an expression without consuming any tokens, and we enter a left recusion without a base case.

This is a common problem with left recursive grammars. Normally, it is solved by refactoring the grammar, but in imperative programming languages a different approach is often used: parsing an expression with _higher precedence_ and using a loop. I first learned about this from [Crafting Interpreters](https://craftinginterpreters.com/parsing-expressions.html#the-parser-class), but it's also covered in [this great blog](https://matklad.github.io/2020/04/13/simple-but-powerful-pratt-parsing.html#Recursive-descent-and-left-recursion) about Pratt parsing.

Speaking of Pratt parsing, the implementation that gets things right has some Pratt vibes (in the sense of parsing precedences). What if don't consider applications when parsing an application? Let's introduce a `term`: an expression that is not an application, but rather an individual "receiver" of the application:

```haskell
term :: Parser LExpr
term = variable <|> abstraction <|> grouping
```

We then obtain

```haskell
expression :: Parser LExpr
expression = application <|> term
```

It is crucial to have `application` before `term`, as we only parse an individual term when `application` fails. (Backtracking issues, yay! But we don't care about performance here.)

Now, let's work on associativity of application. We have to deviate from our grammar here: we can parse a sequence of space-separated terms and build a parse tree with a `fold`. As suggested by the left associativity, we should use `foldl`. In this case, we can use `foldl1` because there's no base case for the usual `foldl`. We will make sure that `terms` is not `[]`.

```haskell
application :: Parser LExpr
application = foldl1 App <$> terms
  where terms = sepBySome term (some $ satisfy isSpace)
```

Observe that by using `sepBySome` instead of `sepBy`, we guarantee that `terms` is nonempty. The constructor `(:)` in the definition of `sepBySome` is the only possible constructor, so we have mathematical guarantees that `sepBySome` will provide a valid input to `foldl1 App` upon injection with `<$>`.

**Exercise.** Simplify the above definition using only `sepBy` and the `Monad` implementation for the parser.

**Another exercise.** There is a redundant expression in our code. Find it and show why it is redundant.

This completes our code. See the full version with comments [here](https://github.com/maleksware/parsers/blob/main/Lambda.hs). As a bonus, let's write a formatter for the parsed expressions:

```haskell
formatExpr :: LExpr -> String
formatExpr expr = case expr of
  Var v -> v
  Abs v rest -> "λ " ++ v ++ ".[" ++ formatExpr rest ++ "]"
  App f a -> "({" ++ formatExpr f ++ "} {" ++ formatExpr a ++ "})"
```

And run this on some expression, say, `(λx.x) t (λy.λz.z)`:

```bash
> formatExpr . fst <$> runParser expression "(λx.x) t (λy.λz.z)"
Just "({({λ x.[x]} {t})} {λ y.[λ z.[z]]})"
```

If the expression is invalid, then the parser will return `Nothing` or leave unconsumed input. When using this parser, check both cases.

