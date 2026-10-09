# Scalars

The following scalar values are supported:

- `Int32` <-> `Int`
- `Float64` <-> `Float`
- `String` <-> `String`
- `Bool` <-> `Boolean`
- `GraphQL::Scalars::ID` <-> `ID`

Built-in custom scalars:

- `GraphQL::Scalars::BigInt` <-> `BigInt`

`ID` accepts strings and integers as input and is always a string in the
response. `BigInt` accepts integer literals of any size as well as strings,
and is returned as a string. An `Int` argument given an integer outside the
32-bit range is answered with an error.

Custom scalars are created by implementing from_json/to_json:

```crystal
@[GraphQL::Scalar]
class ReverseStringScalar < GraphQL::BaseScalar
  @value : String

  def initialize(@value)
  end

  def self.from_json(string_or_io)
    self.new(String.from_json(string_or_io).reverse)
  end

  def to_json(builder : JSON::Builder)
    builder.scalar(@value.reverse)
  end
end
```

`from_json` receives the argument's value serialized as JSON, so a literal
`"abc"` arrives as a JSON string and `4` as a JSON number. A scalar can point
at its specification with [`specified_by_url`](annotations.md#specified_by_url).
