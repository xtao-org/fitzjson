# Other features

This lists features of fitzJSON which whose status is not clear.

Some may be considered for removal, some for addition, most have been experimented with.

These are up for discussion.

## Paths

Note: This is partially implemented, but I'm considering replacing it with specialized decorators to keep the syntax more minimal.

Referencing subvalues in a fitzJSON document by path.

Absolute:

```
top.identifier
top.identifier.path('to').somewhere(0)
```

Relative to parent object/map:

```
self.xyz
```

Referencing environment variables:

```
env.PATH
```

## String-like comments

fitzJSON also allows string-like comments:

```json
/"comment in a string"
```

Raw strings can also be used as comments:

```
/```'comment in a raw string'```
```

```json
/'multiline
string
comment'
```

## Multistrings with arbitrary delimiters

````json
{
  "heredoc": `eof'
  yaba daba
  something something
  'eof`
}
````

## Make single-quoted strings work like JSON strings

Currently single-quote strings are raw multiline strings.

```
'multistring
that spans
multiple lines'
```

Alternatively, they could behave as in ECMAScript -- an alternative to double-quoted strings that supports escape sequences, etc.

```
'regular string\r\n\tindented'
```

This would make fitzJSON more compatible with ECMAScript and derived formats such as JSON5.

Then multistrings would need to be delimited at least by one layer of backticks.

```
`'multistring
that spans lines'`
```

## Pipes

Suffix-based complement to decorators.

```json
{
  "number": 234324239489384238492384823984 | bigint
}
```

## Top-level metadata

```
|{} metadata|
```

## Various specialized decorators

### @merge

For merging maps with maps or lists with lists.

```
@merge [
  [1 2 3]
  [4 5 6]
]
```

### @file

For loading a fitzJSON value from a file

```
@merge [
  // let's say we have a map with some defaults in foo.fitz
  @file 'foo.fitz'

  // and override some of these defaults with new values
  // by merging the new values onto the defaults
  // (producing a new value)
  {
    some_setting: 'non-default value'
  }
]
```