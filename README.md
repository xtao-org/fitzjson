> [!WARNING]
> [EXPERIMENTAL]

This is a little format I've been working on which originated as a hybrid of JSON and a configuration format built on top of a variant of [Jevko](https://jevko.org) (some documentation of early versions [here](http://darius.j.pl/djed.html) and [here](http://darius.j.pl/djedat.html)).

In the design I've tried to address the frequently-complained-about aspects of JSON, add facilities that would make the format pleasant to work with in a text editor (with a primary focus being configuration) as well as introduce a simple but powerful extension mechanism with a familiar syntax. All this while trying to preserve and build further on the self-describing property of JSON. Also in the minimal spirit of JSON I tried to keep the formal definition and spec no less concise than it has to be.

[EXAMPLES.md](EXAMPLES.md)

<p align=center>
<img src="logo2.png" alt="logo" width="160"/>
<h1 align=center>fitzJSON</h1>
</p>

fitzJSON (or "fitz" for short) is a JSON-compatible format optimized for configuration.

fitzJSON is compatible with [JSON](http://www.json.org/) ([ECMA-404](http://www.ecma-international.org/publications/standards/Ecma-404.htm)/[RFC 8259](https://tools.ietf.org/html/rfc8259)) in that a syntactically valid JSON document is also a syntactically valid fitzJSON document.

However, fitzJSON clarifies how numbers and objects should be interpreted by default (in accordance with interoperability guidelines from [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259)) and introduces additional features on top of JSON.

fitzJSON does not attempt to be a strict subset of ECMAScript, though it does happen to overlap with it also outside the JSON core.

 <!-- and so will be semi-compatible with JSON5 et al. Also with edn  -->

## Drop-in replacement for JSON.stringify and JSON.parse

You can easily play with the format today in JavaScript using the [reference implementation](https://github.com/xtao-org/fitzjson.js) of `fitzJSON.parse` and `fitzJSON.stringify` which are meant to be drop-in replacements for `JSON.parse` and `JSON.stringify`.

WARNING: these are still largely experimental and untested. Feedback welcome!

## File extension

The file extension for fitzJSON files should be `.fitz`.

A sensible media type would then be `text/fitz`.

## Features on top of JSON

### Comments

fitzJSON allows C-style single-line and multi-line comments:

```json
// single line
/*
 * multiline
 */
```

### Raw strings

fitzJSON allows multiline raw strings ([multistrings](https://djedr.github.io/posts/multistrings-2023-05-25.html)) which allow verbatim embedding of arbitrary text content (like [heredocs](https://en.wikipedia.org/wiki/Here_document)):

```json
{
  "raw string": 'this string
spans multiple lines
and does not interpret any escape sequences like \n \r \t
'
}
```

🡻

```json
{
  "raw string": "this string\nspans multiple lines\nand does not interpret any escape sequences like \\n \\r \\t\n"
}
```

a raw string may be wrapped in `` ` `` backticks to include an apostrophe:

```json
{
  "raw string": `'this raw string contains an apostrophe: ''`
}
```

🡻

```json
{
  "raw string": "this raw string contains an apostrophe: '"
}
```

To include one or more backticks followed by an apostrophe, the raw string may be wrapped in still more backticks (similarly to how it's done in Markdown):

```json
{
  "raw string": ```'this raw string contains an apostrophe followed by two backticks: '``'```
}
```

🡻

```json
{
  "raw string": "this raw string contains an apostrophe followed by two backticks: '``"
}
```

<details>
The maximum number of backticks that can wrap a string is 255, which should be sufficient for any reasonable (as well as a lot of unreasonable) embeddings of text.
</details>

### Decorators

<!-- fitzJSON takes the self-describing property of JSON to the next level. -->

In fitzJSON, a value may be decorated with zero or more decorators. For example:

```json
{
  "big number": @bigint 3213423435432346456745676567342
}
```

This is a fitzJSON document where the value `3213423435432346456745676567342` is adorned with the `@bigint` decorator.

### Disabled entries and items

fitzJSON offers a simple mechanism for temporarily disabling entries in a map or items in a list, by preceding them with `~`:

```json
{
  ~"not included": 5,
  "array": [~"not included", "included"]
}
```

This is meant to serve as a more convenient (especially when disabling a complex multiline entry/item) and semantic alternative to "commenting out".

### Extended syntax for numbers

On top of the JSON number syntax, fitzJSON adds:

<!-- todo: more on ieee 754 (link?) -->
* `Infinity` and `NaN` as valid numerical values, enabling full support of IEEE 754^[Note: JSON numbers are (purposefully?) [not compatible with IEEE 754](https://xtao.org/blog/json-semantics.html#numbers). In practice this causes interoperability problems.].
* Hexadecimal `0x`, octal `0o`, and binary `0b` number literals.
* Optional `_` digit separators in number literals.

<!-- ```js
seq(
  optional(/[+-]/),
  choice(
    'NaN',
    'Infinity',
    /(0|[1-9](_?[0-9])*)(\.[0-9](_?[0-9])*)?([eE][+-]?[0-9](_?[0-9])*)?/,
    /0[bB][01](_?[01])*/,
    /0[oO][0-7](_?[0-7])*/,
    /0[xX][0-9a-fA-F](_?[0-9a-fA-F])*/,
  )
)
``` -->

### More relaxed syntax

* trailing comma is allowed
* however: commas are optional -- whitespace is sufficient to separate values or entries
* colons to separate keys from values are similarly optional
* keys in objects can be unquoted, provided they look like identifiers
* a fitzJSON document that represents an object can be written without the top-level braces

## Powerful combination of features

### Non-string keys

Decorators can be used to achieve non-string keys in objects/maps. Or turn arrays containing entries into such maps.

### Specialized number types

```
@bigint
@i32

@f64 -- explicit IEEE 754
@u8
```

### Specialized types of various kinds

```
@date
@instant
@uuid
```

### Embedded languages

Multistrings + decorators.

````
someHtml: @html ```'
  <div>
    <p>hello</p>
  </div>
  '```
someMarkdown: @md ```'
  # Title

  Paragraph

  List:

  * item 1
  * item 2
  '```
substitutions: @$ ```'
  Hello, ${name}!
  '```
````

### Expressions

```
@join ['Hello'name'!']
@add [a, b]
@if [cond, conseq, alt]
```

### Decorators powerful enough to implement programming language features

```
factorial: @fn [n 
  @if [
    @eq [n 1] 1 
    @mul [n @call [factorial @sub [n 1]]]
  ]
]
'5!': @call [factorial 5]
```

## Spec

For now [tree-sitter-fitzjson](https://github.com/xtao-org/tree-sitter-fitzjson) is the reference grammar.

[fitzjson.js](https://github.com/xtao-org/fitzjson.js) is semantics (reference interpreter implementation).

[SEMANTICS.md](SEMANTICS.md)

<!-- ## Frictionless editing in a text editor -->

## Similar formats

### JSON5

[JSON5](https://github.com/json5/json5) is a more relaxed syntax for JSON that allows Infinity, NaN, comments, trailing commas, hex numbers, and some other facilities of ECMAScript 5.1. 

fitzJSON is largely (but not entirely, e.g. single-quoted strings are not like in JavaScript) compatible with JSON5 (and even more relaxed).

fitzJSON also brings major features such as multistrings and decorators

### edn

[edn](https://github.com/edn-format/edn) is a subset of Clojure meant for data transfer. A minimal Lisp-like syntax without a strict formal definition and a casual specification. Features extensibility via `#tags`. 

Thanks to making a lot of JSON's punctuation optional, fitzJSON overlaps significantly with edn (some valid edn documents will also be valid fitzJSON). fitzJSON also offers an even more powerful extension mechanism to edn's tags in the form of decorators.

### TOML

[TOML](https://github.com/toml-lang/toml) is a fairly well-supported format for configuration that looks similar to INI. It has a formal definition and a specification. Aims to be minimal, easy, and obvious. 

Features several syntaxes for strings to try and acommodate multiline and raw text, albeit all limited by inflexible delimiters. Also has a convenient number syntax with Infintiy and NaN, hex, octal, and binary literals. Has a native syntax for datetimes.

Comparatively, fitzJSON is no less simple, but more flexible, offering a single syntax for multiline raw strings with flexible delimiters and decorators as a powerful extension mechanism (perhaps intentionally absent in TOML).

### YAML

[YAML](https://github.com/yaml/yaml-spec) is a complex superset of JSON that uses significant indentation for structuring documents. Compared to all listed formats, YAML is highly idiosyncratic and relatively difficult to parse, featuring a detailed specification. It is also fairly widely supported. 

Comparatively fitzJSON is much simpler, without being less flexible, e.g. offering multistrings as a unified syntax for multiline raw strings (instead of multiple specialized syntaxes), and decorators as an even more powerful extension mechanism than YAML's tags.
