# Reference
## events
<details><summary><code>client.Events.QueryEvents() -> *chroniclego.EventListResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write. Results are scoped to the tenant of the API key and ordered newest first by event time and event ID. Pass the opaque `next_cursor` as `cursor` to continue without an offset scan.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.QueryEventsRequest{}
client.Events.QueryEvents(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**source:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**topic:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**eventType:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**entityType:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**entityID:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` — Page size. Values above 200 are clamped to 200.
    
</dd>
</dl>

<dl>
<dd>

**cursor:** `*string` — Opaque position returned as `next_cursor` by the preceding page.
    
</dd>
</dl>

<dl>
<dd>

**since:** `*string` — Relative time window, for example last_7d.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Events.IngestEvent(request) -> *chroniclego.IngestResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:write.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.IngestRequest{
    Source: "my-agent",
    Topic: "conversations",
    EventType: "message.sent",
}
client.Events.IngestEvent(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**request:** `*chroniclego.IngestRequest` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Events.IngestEventBatch(request) -> *chroniclego.IngestResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:write. Maximum 1000 events per batch; larger batches are rejected with 422. Request bodies over the size limit are rejected with 413. Each request consumes 10 rate-limit units.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := []*chroniclego.IngestRequest{
    &chroniclego.IngestRequest{
        Source: "my-agent",
        Topic: "conversations",
        EventType: "message.sent",
    },
}
client.Events.IngestEventBatch(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**request:** `[]*chroniclego.IngestRequest` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Events.StreamEvents() -> chroniclego.EventResult</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write. A Server-Sent Events stream of events matching the optional filters, held open indefinitely.

Opening a stream consumes 5 rate-limit units.

Each message has `event: event` and a `data` field carrying one EventResult as JSON. A comment line arrives every 15 seconds so intermediaries do not close an idle connection.

Every message carries an opaque, stream-specific `id` backed by a monotonic per-tenant delivery sequence. It records ingestion order, independently of the source event's `event_time`. Record the last id you processed and do not parse or construct it.

When `Last-Event-ID` is present, the server first establishes the live subscription, replays matching stored events strictly after that position in ascending order, and then continues with live delivery. Events committed at the history-to-live boundary may be delivered more than once, so consumers should deduplicate by `event_id`. This provides at-least-once delivery across a reconnect without leaving a gap.

Replay is limited to 1000 matching events. An older position returns 409 before the stream opens. Slow consumers are disconnected when the bounded live buffer fills and should reconnect with their last processed id. Concurrent streams are limited per tenant and may return 429 with `Retry-After`.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.StreamEventsRequest{}
client.Events.StreamEvents(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**source:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**eventType:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**entityType:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**entityID:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**lastEventID:** `*string` — Opaque id from the last SSE message the client processed.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## timeline
<details><summary><code>client.Timeline.GetTimeline(EntityType, EntityID) -> *chroniclego.EventPage</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write. Cursor paginated, newest first.

Pass `cursor` from `next_cursor` to read the following page, and stop when `has_more` is false. The cursor is opaque: it is a keyset over `(event_time, event_id)`, it is exclusive so a row cannot repeat across pages, and its encoding may change without notice. Do not parse or construct one.

`include_linked=true` selects a different read that also returns causally linked events. That read is not paginated: it returns one page with `has_more` false, and it cannot be combined with `limit` or `cursor`. `since` is only available on that read, because the paginated read has no time filter.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetTimelineRequest{
    EntityType: "entity_type",
    EntityID: "entity_id",
}
client.Timeline.GetTimeline(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**entityType:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**entityID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` — Page size. Values above the maximum are reduced to it, not rejected.
    
</dd>
</dl>

<dl>
<dd>

**cursor:** `*string` — Opaque cursor from a previous response's next_cursor
    
</dd>
</dl>

<dl>
<dd>

**since:** `*string` — Relative time window, for example last_7d.
    
</dd>
</dl>

<dl>
<dd>

**includeLinked:** `*bool` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## search
<details><summary><code>client.Search.Events(request) -> *chroniclego.EventListResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write. The page size is capped at 200 and a cursor can advance through at most 1,000 relevance-ranked results. Each request consumes 5 rate-limit units.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.SearchRequest{
    Query: "query",
}
client.Search.Events(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**query:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**source:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**entityType:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**entityID:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` 
    
</dd>
</dl>

<dl>
<dd>

**cursor:** `*string` — Opaque position returned by the preceding search page.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## discover
<details><summary><code>client.Discover.ListSources() -> *chroniclego.SourceListResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write. Returns the complete source metadata set without pagination.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
client.Discover.ListSources(
    context.TODO(),
)
```
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Discover.ListEntityTypes() -> *chroniclego.EntityTypeListResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write. Returns the complete entity-type metadata set without pagination.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
client.Discover.ListEntityTypes(
    context.TODO(),
)
```
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Discover.ListEntities(EntityType) -> *chroniclego.EntityListResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write. Entities are ordered by event count and entity ID. The limit is capped at 200; pass `next_cursor` as `cursor`.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListEntitiesRequest{
    EntityType: "entity_type",
}
client.Discover.ListEntities(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**entityType:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` — Page size. Values above 200 are clamped to 200.
    
</dd>
</dl>

<dl>
<dd>

**cursor:** `*string` — Opaque position returned as `next_cursor` by the preceding page.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Discover.GetEventSchema(Source, EventType) -> *chroniclego.SourceSchema</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetEventSchemaRequest{
    Source: "source",
    EventType: "event_type",
}
client.Discover.GetEventSchema(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**source:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**eventType:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## links
<details><summary><code>client.Links.AddEntityRef(request) -> *chroniclego.StatusResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:write.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.AddEntityRefRequest{
    EventID: "event_id",
    EntityType: "entity_type",
    EntityID: "entity_id",
}
client.Links.AddEntityRef(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**eventID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**entityType:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**entityID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**createdBy:** `*string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Links.CreateEventLink(request) -> *chroniclego.CreateLinkResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:write.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateLinkRequest{
    SourceEventID: "source_event_id",
    TargetEventID: "target_event_id",
    LinkType: "link_type",
    Confidence: 1.1,
}
client.Links.CreateEventLink(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**sourceEventID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**targetEventID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**linkType:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**confidence:** `float64` 
    
</dd>
</dl>

<dl>
<dd>

**reasoning:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**createdBy:** `*string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Links.LinkEntities(request) -> *chroniclego.LinkEntityResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:write.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.LinkEntityRequest{
    FromEntityType: "from_entity_type",
    FromEntityID: "from_entity_id",
    ToEntityType: "to_entity_type",
    ToEntityID: "to_entity_id",
}
client.Links.LinkEntities(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**fromEntityType:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**fromEntityID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**toEntityType:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**toEntityID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**createdBy:** `*string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Links.TraverseGraph(request) -> *chroniclego.EventListResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope events:read or events:write. The traversal is bounded by `max_depth`, is not cursor-paginated, and consumes 5 rate-limit units.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GraphRequest{
    StartEventID: "start_event_id",
    Direction: chroniclego.GraphRequestDirectionOutgoing,
}
client.Links.TraverseGraph(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**startEventID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**direction:** `*chroniclego.GraphRequestDirection` 
    
</dd>
</dl>

<dl>
<dd>

**linkTypes:** `[]string` 
    
</dd>
</dl>

<dl>
<dd>

**maxDepth:** `*int` 
    
</dd>
</dl>

<dl>
<dd>

**minConfidence:** `*float64` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## sdk
<details><summary><code>client.Sdk.IdentifyUser(request) -> *chroniclego.AcceptedResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope users:write.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.IdentifyUserRequest{
    UserID: "user_id",
}
client.Sdk.IdentifyUser(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**userID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**traits:** `map[string]any` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Sdk.TrackSignals(request) -> *chroniclego.AcceptedResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope signals:write. Maximum 1000 signals per request; larger batches are rejected with 422. Each request consumes 10 rate-limit units.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.TrackSignalsRequest{}
client.Sdk.TrackSignals(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**signals:** `[]*chroniclego.SignalRequest` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Sdk.TrackTraces(request) -> *chroniclego.AcceptedResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope traces:write. Maximum 1000 traces or total spans per request; larger batches are rejected with 422. Each request consumes 10 rate-limit units.
</dd>
</dl>
</dd>
</dl>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.TrackTracesRequest{}
client.Sdk.TrackTraces(
    context.TODO(),
    request,
)
```
</dd>
</dl>
</dd>
</dl>

#### ⚙️ Parameters

<dl>
<dd>

<dl>
<dd>

**traces:** `[]*chroniclego.TraceRequest` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

