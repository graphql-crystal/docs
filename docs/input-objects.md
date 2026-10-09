# Input Objects

Input objects are objects that are used as field arguments. To define an input
object, use a `GraphQL::InputObject` annotation and inherit `GraphQL::BaseInputObject`.
It must define a constructor with a `GraphQL::Field` annotation.

```crystal
@[GraphQL::InputObject]
class User < GraphQL::BaseInputObject
  getter first_name : String?
  getter last_name : String?

  @[GraphQL::Field]
  def initialize(@first_name : String?, @last_name : String?)
  end
end
```

Every constructor argument needs a type restriction. Nilable arguments and
arguments with a default value are optional input fields; the rest are
required, and a request that omits one or passes `null` for it is answered
with an error. Unknown fields in the input are reported as well.

The constructor's `arguments` option names, describes or deprecates input
fields exactly as it does for field arguments, see
[Annotation arguments](annotations.md#arguments).
