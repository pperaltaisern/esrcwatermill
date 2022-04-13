# esrcwatermill
Package that enable CQRS to esrc using watermill (https://github.com/ThreeDotsLabs/watermill).

```go
import (
	"github.com/pperaltaisern/esrc/relay"
	"github.com/ThreeDotsLabs/watermill/components/cqrs"
)

type Publisher struct {
	bus *cqrs.EventBus
}

var _ relay.Publisher = (*Publisher)(nil)
```

## How to

In esrc, relayed events are already marshaled. For this reason, the EventBus must be instanced with this package's custom marshaler:

```go
type RelayEventMarshaler struct {
	NewUUID      func() string
	CmdMarshaler cqrs.CommandEventMarshaler
}

var _ cqrs.CommandEventMarshaler = (*RelayEventMarshaler)(nil)
```

It will use a custom marshal process for events, but will rely on another marshaler for commands.

## For performance enhancement

Having the aggregate ID marshaled and stored in the event store in every event is redundant (an event belongs to an aggregate). To avoid this, make sure your events doesn't marshal the aggregate ID and implement this package event's interface:

```go
type Event interface {
	WithAggregateID(id string)
}
```

Example:

```go
type MyAggregateCreatedEvent struct {
	AggregateID uuid.UUID `json:"-"`
}

func (e *MyAggregateCreatedEvent) WithAggregateID(id string) {
	e.AggregateID = UUIDFromString(id)
}
```