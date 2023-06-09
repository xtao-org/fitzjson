# Other features

This lists features of fitzJSON which whose status is not clear.

Some may be considered for removal, some for addition, most have been experimented with.

These are up for discussion.

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