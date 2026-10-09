# Upgrading from 0.4

Releases 0.5 through 0.9 fixed the executor where it strayed from the GraphQL
specification and added interfaces, unions and subscriptions. Most code
written against 0.4 keeps compiling unchanged. What clients see did change
in a few places, all of them required by the specification.

## Responses

- A field whose resolver raised is `null` instead of being left out of the
  response, and when that field is non-null the `null` propagates upwards,
  so `"data"` itself can be `null`. See [Errors](errors.md).
- Errors on fields carry `locations` with the line and column in the query.
- Request-level errors, such as a syntax error or a missing variable, no
  longer carry an empty `path`.
- Queries that 0.4 silently accepted are now rejected: unknown arguments,
  unknown input fields, unknown directives, selection sets on scalars, and
  object fields without a selection set.
- Running a subscription through `schema.execute` is answered with an error;
  use `schema.subscribe`.

## Variables and arguments

- Omitting a nullable variable, passing `null`, or relying on a default
  declared in the operation now works. In 0.4 any omitted variable failed
  the request.
- Argument names set through the `arguments` option are now honored when
  resolving; in 0.4 they only appeared in the schema.
- A single value passed where a list is expected becomes a one-element list.
- Integer literals that do not fit `Int32` reach custom scalars such as
  `BigInt`; `Int` arguments reject them.

## API

- `GraphQL::Language.parse` takes a `max_depth` argument, and the unused
  `Parser#max_nesting` property is gone.
- `GraphQL::Introspection::Schema.new` takes the subscription type as a
  fourth argument.
- `GraphQL::Context` gained `max_concurrency`, `max_complexity`,
  `complexity` and `max_depth`. `max_complexity` and `complexity` existed
  before but were never read.
- `GraphQL::Schema.new` accepts a subscription type as its third argument.

## Behavior

- Fields and array elements resolve sequentially unless
  `max_concurrency` is set; earlier builds spawned one fiber per field.
  Root mutation fields always resolve one after another.
- Queries nested more than 100 levels are rejected while parsing; raise
  `max_depth` on the context if you really need deeper ones.

The [changelog](https://github.com/graphql-crystal/graphql/blob/main/CHANGELOG.md)
lists every change by release.
