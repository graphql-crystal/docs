# Objects

Objects are perhaps the most commonly used type in GraphQL. They are implemented
as classes. To define a object, we need a `GraphQL::Object` annotation and to inherit
`GraphQL::BaseObject`. Fields are methods with a `GraphQL::Field` annotation.

```crystal
@[GraphQL::Object]
class Foo < GraphQL::BaseObject
  # type restrictions are mandatory on fields
  @[GraphQL::Field]
  def hello(first_name : String, last_name : String) : String
    "Hello #{first_name} #{last_name}"
  end

  # besides basic types, we can also return other objects
  @[GraphQL::Field]
  def bar : Bar
    Bar.new
  end
end

@[GraphQL::Object]
class Bar < GraphQL::BaseObject
  @[GraphQL::Field]
  def baz : Float64
    42_f64
  end
end
```

For simple objects, we can use instance variables:

```crystal
@[GraphQL::Object]
class Foo < GraphQL::BaseObject
  @[GraphQL::Field]
  property bar : String

  @[GraphQL::Field]
  getter baz : Float64

  def initialize(@bar, @baz)
  end
end
```

A nilable return type such as `String?` becomes a nullable field; any other
type is non-null. Arrays become lists, and the element type follows the same
rule: `Array(Bar)` is `[Bar!]!`, `Array(Bar?)` is `[Bar]!`.

## Query

Query is the root type of all queries.

```crystal
@[GraphQL::Object]
class Query < GraphQL::BaseQuery
  @[GraphQL::Field]
  def echo(str : String) : String
    str
  end
end

schema = GraphQL::Schema.new(Query.new)
```

## Mutation

Mutation is the root type for all mutations.

```crystal
@[GraphQL::Object]
class Mutation < GraphQL::BaseMutation
  @[GraphQL::Field]
  def echo(str : String) : String
    str
  end
end

schema = GraphQL::Schema.new(Query.new, Mutation.new)
```

The root fields of a mutation run one after another, in the order the
operation lists them.

## Field Arguments

Field arguments are automatically resolved. A type with a default value becomes
optional. A nilable type is also considered a optional type.

```crystal
@[GraphQL::Field]
def greet(name : String = "world", title : String? = nil) : String
  title ? "Hello, #{title} #{name}!" : "Hello, #{name}!"
end
```

Arguments may be given inline or through variables. Variables follow the
specification: a nullable variable may be omitted or set to `null`, a default
declared on the operation (`query ($n: Int = 1)`) applies when the variable is
not sent, and omitting a non-null variable is an error. A single value given
where a list is expected is coerced to a one-element list.

A field may also declare a parameter typed as your [context](context.md)
class; it is filled with the current context and does not appear in the
schema.
