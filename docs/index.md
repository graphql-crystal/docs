![Logo](https://raw.githubusercontent.com/graphql-crystal/graphql/main/assets/logo.svg)

GraphQL server library for Crystal.

- **Boilerplate-free**: Schema generated at compile time
- **Type-safe**: Crystal guarantees your code matches your schema
- **High performance**: See [benchmarks](https://github.com/graphql-crystal/benchmarks)

## Getting Started

Install the shard by adding the following to our `shard.yml`:

```yaml
dependencies:
  graphql:
    github: graphql-crystal/graphql
```

Then run `shards install`. Crystal 1.4.0 or later is required.

The first step is to define a query object. This is the root type for all
queries and it looks like this:

```crystal
require "graphql"

@[GraphQL::Object]
class Query < GraphQL::BaseQuery
  @[GraphQL::Field]
  def hello(name : String) : String
    "Hello, #{name}!"
  end
end
```

Now we can create a schema object:

```crystal
schema = GraphQL::Schema.new(Query.new)
```

To verify we did everything correctly, we can print out the schema:

```crystal
puts schema.document.to_s
```

Which, among several built-in types, prints our query type:

```graphql
type Query {
  hello(name: String!): String!
}
```

To serve our API over HTTP we call `schema.execute` with the request parameters and receive a JSON string. Here is an example for Kemal:

```crystal
post "/graphql" do |env|
  env.response.content_type = "application/json"

  query = env.params.json["query"].as(String)
  variables = env.params.json["variables"]?.as(Hash(String, JSON::Any)?)
  operation_name = env.params.json["operationName"]?.as(String?)

  schema.execute(query, variables, operation_name)
end
```

Now we're ready to query our API:

```bash
curl \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{ "query": "{ hello(name: \"John Doe\") }" }' \
  http://0.0.0.0:3000/graphql
```

This should return:

```json
{ "data": { "hello": "Hello, John Doe!" } }
```

For easier development, we recommend using [GraphiQL](https://github.com/graphql/graphiql).
A starter template combining Kemal, GraphiQL and a live subscription is found at
[examples/graphiql](https://github.com/graphql-crystal/graphql/tree/main/examples/graphiql).

## Where to go next

- [Objects, queries and mutations](objects.md) for the types that make up a schema
- [Context](context.md) for passing request data to fields and limiting what a query may do
- [Subscriptions](subscriptions.md) for streaming responses over WebSockets
- [Errors](errors.md) for how failures appear in a response
- [Upgrading from 0.4](upgrading.md) if you are coming from an earlier release
