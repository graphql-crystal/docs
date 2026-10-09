# Subscriptions

A subscription type inherits `GraphQL::BaseSubscription`. Its fields return a
`Channel` whose element type is the field's GraphQL type, and every value sent
on the channel becomes one response. `GraphQL::Broadcast` fans values out to
any number of subscribers, which is the usual way to publish from a mutation:

```crystal
MESSAGES = GraphQL::Broadcast(Message).new

@[GraphQL::Object]
class Mutation < GraphQL::BaseMutation
  @[GraphQL::Field]
  def post(text : String) : Message
    message = Message.new(text)
    MESSAGES.publish(message)
    message
  end
end

@[GraphQL::Object]
class Subscription < GraphQL::BaseSubscription
  @[GraphQL::Field]
  def message_added : Channel(Message)
    MESSAGES.subscribe
  end
end

schema = GraphQL::Schema.new(Query.new, Mutation.new, Subscription.new)
```

`schema.subscribe` takes the same arguments as `schema.execute` and returns a
`GraphQL::Subscription` that yields response documents. It ends when the
resolver's channel closes. Call `close` to unsubscribe, which also closes the
resolver's channel:

```crystal
subscription = schema.subscribe(%(subscription { messageAdded { text } }))
spawn do
  subscription.each do |response|
    puts response # {"data":{"messageAdded":{"text":"hi"}}}
  end
end
```

A subscription operation must select exactly one root field. A request that
cannot be started, for example because it selects two root fields, yields a
single error response and is already closed. Running a subscription through
`schema.execute` is answered with an error pointing at `schema.subscribe`.

Each event is rendered like a query result: field errors, `null` for failed
fields and `locations` all work per event, see [Errors](errors.md). The
context passed to `schema.subscribe` is reused for every event.

## Broadcast

`GraphQL::Broadcast(T)` keeps one buffered channel per subscriber.
`subscribe` returns a new channel, `publish` sends a value to every
subscriber, and `unsubscribe` or closing the channel removes it. Every
subscriber channel buffers 16 values by default (`Broadcast(T).new(buffer)`
changes that). A subscriber that has not drained its buffer misses the next
value rather than blocking the publisher and every other subscriber.

## Over WebSockets

`GraphQL::Transport::WebSocket` speaks the `graphql-transport-ws` protocol
used by graphql-ws, Apollo Client, urql and Relay. It is not loaded by
`require "graphql"`. With Kemal:

```crystal
require "graphql/transport/ws"

ws "/graphql" do |socket, env|
  GraphQL::Transport::WebSocket.new(schema, socket) { MyContext.new(env) }
end
```

Browsers insist that the server echoes the `graphql-transport-ws` subprotocol
during the handshake, which Kemal's `ws` helper cannot do. For browser
clients, mount the standard library handler instead and pass the protocol
name so the handshake negotiates it. The subprotocol argument needs Crystal
1.20 or later. With Kemal's path-scoped `use`, requests that are not upgrades
fall through to the HTTP route on the same path:

```crystal
subscriptions = HTTP::WebSocketHandler.new([GraphQL::Transport::WebSocket::PROTOCOL]) do |socket, http|
  GraphQL::Transport::WebSocket.new(schema, socket) { MyContext.new(http.request) }
end
use "/graphql", subscriptions
```

The block builds the context for every operation on the connection. Queries
and mutations sent over the socket are executed once and answered with
`next` and `complete`. Subscriptions stream until the client sends `complete`
or disconnects.

The [GraphiQL example](https://github.com/graphql-crystal/graphql/tree/main/examples/graphiql)
in the repository wires all of this up with a live subscription in the
browser.
