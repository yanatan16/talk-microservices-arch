<style>
img { height: 400px; }
img[alt=tyson-love], img[alt=tyson-field], img[alt=tyson-sleep] { height: 300px; }
hr { margin: 2em 0; }
</style>

# Parser-Combinators in Javascript
## Tyson's journey

_Jon Eisen_

BoulderJS Meetup

February 17, 2016

http://joneisen.me/talk-js-parser

-***-

# Background

In November, Jim Baker presented about writing a Javascript Interpreter in Javascript.

-^*^-

# Background

Many discussed how interesting it would be to look at parsing Javascript.

-^*^-

# Background

I said "**That's not as approachable!**"

I'll talk about parser-combinators instead.

-***-

## Our Hero

![tyson-cold](img/tyson-cold.jpg)

Tyson wants to create a new language.

-***-

## But Parsing is Hard!

![tyson-dinosaur](img/tyson-dinosaur.jpg)

Tyson doesn't even know where to start.

-***-

## So Boring

![tyson-sleep](img/tyson-sleep.jpg)

Lexers? Context-free Grammars? Backus-Naur Form?

Do bison even _have_ ANTLRs?

Its so complicated and academic!

-***-

## But Wait!

![tyson-sweater](img/tyson-sweater.jpg)

Parser Combinators To the Rescue!

-***-

## Parser Combinators

- A parsing grammar that is also **runnable code**
- Readable, modular, and testable components
- Technically: A recursive descent parser

---

- First introduced in 1989 by [Frost and Launchbury](http://richard.myweb.cs.uwindsor.ca/PUBLICATIONS/COMPJ_89.pdf)
- First implementations
    - Published by [Frost, Hafiz, and Callaghan in 2008](http://richard.myweb.cs.uwindsor.ca/PUBLICATIONS/PADL_08.pdf)
    - [attoparsec](https://github.com/bos/attoparsec) (Haskell) in 2007
    - [parsec](https://github.com/aslatter/parsec) (Haskell) in 2008

-***-

## Intriguing...

![tyson-stairs](img/tyson-stairs.jpg)

Tyson patiently waits for an example.

-***-

## Small Examples

```js
var P = require('parsimmon')

var number = P.regex(/[0-9]+/).map(parseInt)
var list = P.sepBy(number, P.string(','))

list.parse('1,2,3,4') // => [1,2,3,4]
```

We can parse numbers and lists.

---

```js
var string = P.regex(/"([^"]*)"/, 1)

string.parse('"abc"').value // => "abc"
```

We can parse strings.

-^*^-

## AST Node Example

```js
function lexeme(p) { return p.skip(P.optWhitespace) }

var op = lexeme(P.oneOf('+-/*'))
var number_ = lexeme(number)

var expr = P.seq(number_, op, number_).map(function (arr) {
  return { type: 'binary-op',
           op: arr[1],
           left: arr[0],
           right: arr[2] }
})

expr.parse('5 + 3').value
// => { type: 'binary-op', op: '+', left: 5, right: 3 }
```

We can parse expressions and create an AST node.

-^*^-

## Recursive Parsing Example

```js
var LB = lexeme(P.string('['))
var RB = lexeme(P.string(']'))
var COMMA = lexeme(P.string(','))
var literal = P.alt(number_, string)

var expr = P.lazy(function () {
  return P.alt(array, literal)
})

var array = LB.then(P.sepBy(expr, COMMA)).skip(RB)

expr.parse('["fail"') // => fails
expr.parse('[]').value // => []
expr.parse('[1, 2, "abc"]').value // => [1,2,"abc"]
expr.parse('[1, [1], [[]]]').value // => [1, [1], [[]]]
```

We can parse recursive structures.

-***-

# Whoa

![tyson-yoga](img/tyson-yoga.jpg)

Tyson is amazed at how simple it is!

-***-

## A Trek Starts

![tyson-hike](img/tyson-hike.jpg)

Let's parse a language, lisp!

-***-

## Parsing Lisp

```js
const ignore = regex(/(\s*(;[^\n]*(\n\s*|$))?)*/)
function lexeme(p) { return p.skip(ignore) }
```

Start with what we ignore: Whitespace and comments.

Comments are `; semicolon-based`

-^*^-

## Parsing Lisp

```js
const lparen = lexeme(string('(')),
      rparen = lexeme(string(')'))
```

The basics of lisp are parentheses.

---

```js
const symbol = regex(/[a-z_][a-z0-9-_]*/i),
      quoted = regex(/"((?:\\.|.)*?)"/, 1),
      numeral = regex(/-?(0|[1-9]\d*)([.]\d+)?(e[+-]?\d+)?/i)
```

We handle symbols, strings, and numbers as literals.

-^*^-

## Parsing Lisp

```js
const form = lazy('a lisp form', function () {
    return alt(        // order of specificity of first char
        sexp,          // begins with (
        stringLiteral, // begins with "
        numberLiteral, // begins with [-.\d]
        symbolForm)    // begins with [a-zA-Z-_]
})
```

A recursive form can contain one:

- an s-expression
- a string literal
- a number literal
- a symbol

-^*^-

## Parsing Lisp

```js
const forms = lexeme(form).many()
      sexp = lparen.then(forms).skip(rparen)
```

S-expressions are parenthetically-wrapped forms. It returns a JS array.

---

```js
const stringLiteral = quoted.map(util.interpretJsonEscapes),
      numberLiteral = numeral.map(parseFloat),
      symbolForm = symbol.map(function (x) { return { sym: x }})
```

- JSON quoting rules are applied to strings.
- Numbers are parsed into integers or floats.
- Symbols return a small object.

-^*^-

## Parsing Lisp

```js
// Empty forms
> lisp('()')
{ status: true, value: [] }
```
```js
// failure
> lisp('((((abc')
{ status: false, index: 7, expected: [ 'an s-expression' ] }
```
```js
// symbols
> lisp('(foo bar)').value
[ { sym: 'foo' }, { sym: 'bar' } ]
```
```js
// strings
> lisp('("foo" "bar")').value
[ 'foo', 'bar' ]
```
```js
// numbers
> lisp('(5 plus 6)').value
[ 5, { sym: 'plus' }, 6 ]
```
```js
// comments
> lisp('(i handle ; comments\n)').value
[ { sym: 'i' }, { sym: 'handle' } ]
```

-^*^-

## Parsing Lisp

Now we can take the parsed Abstract Syntax Tree, and do stuff with it!

... But thats for another talk, specifically [Jim Baker's November BoulderJS Talk](http://www.meetup.com/Boulder-JS/events/226663066/)

-***-

# Freedom!

![tyson-field](img/tyson-field.jpg)

With this newfound knowledge, Tyson runs free to pursue his parsing dreams.

-***-

# References

- [Wikipedia entry for Parser-Combinators](https://en.wikipedia.org/wiki/Parser_combinator)
- [Parsimmon JS Parsing Library](https://github.com/jneen/parsimmon)
- [Using Parsec, from Real World Haskell](http://book.realworldhaskell.org/read/using-parsec.html)

More examples available in this repo:

http://github.com/yanatan16/talk-js-parser/tree/gh-pages/lib

-***-

## Thanks

![tyson-love](img/tyson-love.jpg)

Jon Eisen is a local freelancer.

Contact if you want to hire him (or pet Tyson).

http://joneisen.works
