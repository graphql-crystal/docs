# Enums

Defining enums is straightforward. Just add a `GraphQL::Enum` annotation:

```crystal
@[GraphQL::Enum]
enum IPAddressType
  IPv4
  IPv6
end
```

Enum values are accepted as enum literals (`IPv4`) and, through variables, as
strings. Members can be described or deprecated with the
[`values`](annotations.md#values) option.
