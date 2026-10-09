# Annotation Arguments

## name

Supported on: `Object`, `InputObject`, `Field`, `Enum`, `Scalar`, `Interface`, `Union`

We can use the `name` argument to customize the introspection type name of a
type. This is not needed in most situations because type names are automatically
converted to PascalCase or camelCase. However, `item_id` converts to
`itemId`, but we might want to use `itemID`. For this, we can use the `name`
argument.

```crystal
@[GraphQL::Object(name: "Greeter")]
class GreetingService
  @[GraphQL::Field(name: "hello")]
  def say_hello : String
    "Hello!"
  end
end
```

## description

Supported on: `Object`, `InputObject`, `Field`, `Enum`, `Scalar`, `Interface`, `Union`

Describes the type. Descriptions are available through the introspection interface
so it's always a good idea to set this argument.

```crystal
@[GraphQL::Object(description: "Produces greetings in several languages.")]
class Greeter
end
```

## deprecated

Supported on: `Field`

The deprecated argument marks a field as deprecated.

```crystal
class Greeter
  @[GraphQL::Field(deprecated: "Use greet(lang:) instead.")]
  def hello : String
    "Hello!"
  end
end
```

Arguments, input object fields and enum values are deprecated through the
`arguments` and `values` options below.

## arguments

Supported on: `Field`

Sets names, descriptions and deprecations for field arguments, and for the
input fields of an input object when used on its constructor. Each argument
may set `name`, `description` and `deprecated`:

```crystal
class Greeter
  @[GraphQL::Field(arguments: {
    language: {name: "lang", description: "Language code of the greeting"},
    formal:   {deprecated: "Greetings are always informal now"},
  })]
  def greet(language : String, formal : Bool? = nil) : String
    case language
    when "fr" then "Bonjour"
    when "de" then "Hallo"
    else           "Hello"
    end
  end
end
```

Deprecated arguments and input fields are hidden from introspection unless
`includeDeprecated: true` is passed.

## values

Supported on: `Enum`

Describes or deprecates enum members by constant name:

```crystal
@[GraphQL::Enum(values: {
  Red:  {description: "Like a rose"},
  Blue: {deprecated: "Use Navy"},
})]
enum Color
  Red
  Blue
  Navy
end
```

## specified_by_url

Supported on: `Scalar`

Points at the specification of the scalar's format and is reported as
`specifiedByURL` in introspection:

```crystal
@[GraphQL::Scalar(specified_by_url: "https://tools.ietf.org/html/rfc3339")]
record DateTime, value : Time do
  # ...
end
```
