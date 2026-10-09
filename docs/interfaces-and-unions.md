# Interfaces and Unions

## Interfaces

An interface is an abstract class or a module with a `GraphQL::Interface`
annotation. Its fields are `GraphQL::Field` methods, abstract or not. Objects
that inherit from the class or include the module implement the interface,
and fields may return the interface type:

```crystal
@[GraphQL::Interface]
abstract class Character < GraphQL::BaseObject
  @[GraphQL::Field]
  abstract def name : String
end

@[GraphQL::Object]
class Human < Character
  @[GraphQL::Field]
  def name : String
    "Luke"
  end

  @[GraphQL::Field]
  def home_planet : String
    "Tatooine"
  end
end

@[GraphQL::Object]
class Query < GraphQL::BaseQuery
  @[GraphQL::Field]
  def hero : Character
    Human.new
  end
end
```

Every object implementing an interface is part of the schema as soon as the
interface is. Fields specific to one implementation are selected through
fragments with a type condition, and `__typename` reports the concrete type:

```graphql
{
  hero {
    __typename
    name
    ... on Human {
      homePlanet
    }
  }
}
```

A fragment conditioned on the interface itself (`... on Character`) applies
to every implementation, and one conditioned on a type the object is not is
skipped. Introspection lists the interfaces of an object and the
`possibleTypes` of an interface.

## Unions

A union is a module with a `GraphQL::Union` annotation. Objects that include
the module are its member types. A union has no fields of its own, so
selections on it must go through type conditions:

```crystal
@[GraphQL::Union]
module SearchResult
end

@[GraphQL::Object]
class Human < GraphQL::BaseObject
  include SearchResult
  # ...
end

@[GraphQL::Object]
class Starship < GraphQL::BaseObject
  include SearchResult
  # ...
end

@[GraphQL::Object]
class Query < GraphQL::BaseQuery
  @[GraphQL::Field]
  def search(text : String) : Array(SearchResult)
    [Human.new, Starship.new] of SearchResult
  end
end
```

```graphql
{
  search(text: "a") {
    __typename
    ... on Human { name }
    ... on Starship { length }
  }
}
```

Both annotations accept `name` and `description` like `GraphQL::Object`.
