# Context

`context` is a optional argument that our fields can retrieve. It lets fields
access global data, like database connections.

```crystal
# Define our own context type
class MyContext < GraphQL::Context
  @pi : Float64
  def initialize(@pi)
  end
end

# Pass it to schema.execute
context = MyContext.new(Math::PI)
schema.execute(query, variables, operation_name, context)

# Access it in our fields
@[GraphQL::Object]
class MyMath < GraphQL::BaseObject
  @[GraphQL::Field]
  def pi(context : MyContext) : Float64
    context.pi
  end
end
```

Context instances must not be reused for multiple executions. A subscription
is one execution: the context passed to `schema.subscribe` is kept for the
lifetime of the subscription and reused for every event it delivers, and the
WebSocket transport builds one context per operation on a connection. A
resolver that reads per-request state from the context therefore sees the
same values for as long as the subscription runs.

## Handling exceptions

When a resolver raises, the context decides what happens. `handle_exception`
receives the exception and returns the message to put in the response's
`errors`; the default returns `ex.message`. Return `nil` to report nothing,
or raise to let the exception bubble out of `schema.execute`:

```crystal
class MyContext < GraphQL::Context
  def handle_exception(ex : ::Exception) : String?
    ex.is_a?(NotFound) ? "not found" : raise ex
  end
end
```

How a failed field appears in the response is described under
[Errors](errors.md).

## Concurrency

By default, a query is resolved sequentially in the fiber that called
`schema.execute`. To resolve fields and array elements concurrently, set
`max_concurrency` on the context:

```crystal
context = MyContext.new(Math::PI)
context.max_concurrency = 8
schema.execute(query, variables, operation_name, context)
```

This is the maximum number of fibers a single execution may have running at
once. When the budget is used up, remaining work runs inline, so a large
list never fans out into an unbounded number of fibers. Choose a value your
downstream resources can sustain; resolvers that check out database
connections should stay below the connection pool size.

The root fields of a mutation are always resolved one after another, as the
specification requires; fields nested below them still run concurrently.

## Complexity

To reject operations that select too many fields, set `max_complexity` on the
context:

```crystal
context = MyContext.new(Math::PI)
context.max_complexity = 200
schema.execute(query, variables, operation_name, context)
```

Complexity is the number of fields the operation selects, counted across the
whole selection tree with fragments expanded. It is computed before any
resolver runs, and an operation over the limit is answered with an error and
no data. After execution, `context.complexity` holds the count.

## Depth

Nesting depth is limited separately. Selection sets, lists and input objects
may nest `max_depth` levels, 100 by default, and a deeper query is rejected
while parsing. Real queries rarely pass twenty levels; the limit exists so a
hostile query cannot exhaust the stack:

```crystal
context.max_depth = 30
```
