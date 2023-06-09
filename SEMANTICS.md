Note: I am still working out many details here and organizing my thoughts.

A major issue is that I conflate specialization of fitzJSON for configuration with specialization for data interchange.

All these are fuzzy terms. Most likely data interchange formats and configuration formats overlap significantly. Most features of a data interchange format are necessary for configuration. Some are ok. It's probably perfectly fine if a data interchange format is used as a basis for a configuration format.

Anyway, I digress.

# fitzJSON semantics

## Numbers

For undecorated numbers, it is recommended to follow the guidelines from [RFC 8259, section 6.](https://datatracker.ietf.org/doc/html/rfc8259#section-6):

> This specification allows implementations to set limits on the range
> and precision of numbers accepted.  Since software that implements
> IEEE 754 binary64 (double precision) numbers [IEEE754] is generally
> available and widely used, good interoperability can be achieved by
> implementations that expect no more precision or range than these
> provide, in the sense that implementations will approximate JSON
> numbers within the expected precision.  A JSON number such as 1E400
> or 3.141592653589793238462643383279 may indicate potential
> interoperability problems, since it suggests that the software that
> created it expects receiving software to have greater capabilities
> for numeric magnitude and precision than is widely available.

> Note that when such software is used, numbers that are integers and
> are in the range [-(2**53)+1, (2**53)-1] are interoperable in the
> sense that implementations will agree exactly on their numeric
> values.

So: don't expect the consumer of your unadorned fitzJSON number to interpret it with greater precision than IEEE 754.

If you require greater or some specific precision or number format, use a decorator, like:

```
@i32 1234557
@u8 234
@bigint 3248493458643268732659723459324654732985
@bignum 3.141592653589793238462643383279
```

etc.

Make sure that both sides are clear as to the meaning of the decorators. If you care about correctness, don't ignore decorators -- error on unrecognized ones and find (or write) an implementation for them that is in line with the agreement between producer and consumer about the required semantics.

As per [ECMA-404](https://www.ecma-international.org/wp-content/uploads/ECMA-404_2nd_edition_december_2017.pdf):

> The JSON syntax is not a specification of a complete data interchange. Meaningful data interchange requires agreement between a producer and consumer on the semantics attached to a particular use of the JSON syntax. What JSON does provide is the syntactic framework to which such semantics can be attached.

fitzJSON is also not a specification of a complete data interchange. It builds on top of JSON to clarify certain aspects based on experience and provides additional syntactic tools (a larger framework) for making data more self-describing. This is to allow reifying some of the agreement between producer and consumer as part of the data being exchanged.

## Objects

[RFC 8259, section 4.](https://datatracker.ietf.org/doc/html/rfc8259#section-4):

> An object whose names are all unique is interoperable in the sense
> that all software implementations receiving that object will agree on
> the name-value mappings.  When the names within an object are not
> unique, the behavior of software that receives such an object is
> unpredictable.  Many implementations report the last name/value pair
> only.  Other implementations report an error or fail to parse the
> object, and some implementations report all of the name/value pairs,
> including duplicates.

> JSON parsing libraries have been observed to differ as to whether or
> not they make the ordering of object members visible to calling
> software.  Implementations whose behavior does not depend on member
> ordering will be interoperable in the sense that they will not be
> affected by these differences.