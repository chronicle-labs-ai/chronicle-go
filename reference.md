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

## agents
<details><summary><code>client.Agents.ListAgents() -> []*chroniclego.AgentSummary</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
client.Agents.ListAgents(
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

<details><summary><code>client.Agents.SearchAgentHashIndex() -> []*chroniclego.HashIndexEntry</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.SearchAgentHashIndexRequest{}
client.Agents.SearchAgentHashIndex(
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

**q:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**domains:** `*string` — Comma-separated hash domains.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Agents.SubscribeToAgentChanges() -> map[string]any</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
client.Agents.SubscribeToAgentChanges(
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

<details><summary><code>client.Agents.UpdateAgent(Name, request) -> *chroniclego.AgentSummary</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.UpdateAgentRequest{
    Name: "name",
}
client.Agents.UpdateAgent(
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

**name:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**environment:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**owner:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**purpose:** `*string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Agents.GetAgentSnapshot(Name) -> *chroniclego.AgentSnapshot</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetAgentSnapshotRequest{
    Name: "name",
}
client.Agents.GetAgentSnapshot(
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

**name:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Agents.PinLatestAgentVersion(Name) -> *chroniclego.AgentSummary</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.PinLatestAgentVersionRequest{
    Name: "name",
}
client.Agents.PinLatestAgentVersion(
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

**name:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Agents.CreateAgentChatSession(Name) -> *chroniclego.CreateAgentChatSessionResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateAgentChatSessionRequest{
    Name: "name",
}
client.Agents.CreateAgentChatSession(
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

**name:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Agents.GetAgentChatSession(Name, SessionID) -> *chroniclego.AgentChatSession</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetAgentChatSessionRequest{
    Name: "name",
    SessionID: "session_id",
}
client.Agents.GetAgentChatSession(
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

**name:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**sessionID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Agents.SendAgentChatMessage(Name, SessionID, request) -> *chroniclego.SendAgentChatMessageResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.SendAgentChatMessageRequest{
    Name: "name",
    SessionID: "session_id",
    Text: "text",
}
client.Agents.SendAgentChatMessage(
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

**name:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**sessionID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**text:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Agents.RegisterAgentArtifact(request) -> *chroniclego.AgentVersionSummary</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope agents:write.
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
request := &chroniclego.RegisterAgentArtifactRequest{
    Artifact: &chroniclego.RegisterAgentArtifactRequestArtifact{
        ArtifactID: "artifactId",
        ConfigHash: "configHash",
        Framework: chroniclego.RegisterAgentArtifactRequestArtifactFrameworkVercelAiSdk,
        Model: &chroniclego.RegisterAgentArtifactRequestArtifactModel{
            Label: "label",
        },
        Name: "name",
        Provenance: &chroniclego.RegisterAgentArtifactRequestArtifactProvenance{
            CreatedAt: chroniclego.MustParseDateTime(
                "2024-01-15T09:30:00Z",
            ),
        },
        SchemaVersion: "schemaVersion",
        Tools: []*chroniclego.RegisterAgentArtifactRequestArtifactToolsItem{
            &chroniclego.RegisterAgentArtifactRequestArtifactToolsItem{
                Name: "name",
            },
        },
        Version: "version",
    },
}
client.Agents.RegisterAgentArtifact(
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

**artifact:** `*chroniclego.RegisterAgentArtifactRequestArtifact` 
    
</dd>
</dl>

<dl>
<dd>

**metadata:** `*chroniclego.RegisterAgentArtifactRequestMetadata` — Mutable, human-authored metadata attached to a logical Agent identity. Artifact configuration remains immutable inside `AgentRegistryVersionRecord`.
    
</dd>
</dl>

<dl>
<dd>

**status:** `*chroniclego.RegisterAgentArtifactRequestStatus` — Defaults to `current`. Registering a new current version atomically demotes the previous current version to stable.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Agents.RecordAgentRuns(request) -> *chroniclego.RecordAgentRunsResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Requires scope agents:write.
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
request := &chroniclego.RecordAgentRunsRequest{
    Runs: []*chroniclego.RecordAgentRunsRequestRunsItem{
        &chroniclego.RecordAgentRunsRequestRunsItem{
            ArtifactID: "artifactId",
            ConfigHash: "configHash",
            Operation: chroniclego.RecordAgentRunsRequestRunsItemOperationGenerate,
            RunID: "runId",
            SchemaVersion: "schemaVersion",
            StartedAt: chroniclego.MustParseDateTime(
                "2024-01-15T09:30:00Z",
            ),
            Status: chroniclego.RecordAgentRunsRequestRunsItemStatusStarted,
            ToolCalls: []*chroniclego.RecordAgentRunsRequestRunsItemToolCallsItem{
                &chroniclego.RecordAgentRunsRequestRunsItemToolCallsItem{
                    CallID: "callId",
                    StartedAt: chroniclego.MustParseDateTime(
                        "2024-01-15T09:30:00Z",
                    ),
                    Status: chroniclego.RecordAgentRunsRequestRunsItemToolCallsItemStatusStarted,
                    ToolName: "toolName",
                },
            },
        },
    },
}
client.Agents.RecordAgentRuns(
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

**runs:** `[]*chroniclego.RecordAgentRunsRequestRunsItem` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## datasets
<details><summary><code>client.Datasets.ListDatasets() -> *chroniclego.TaskSuitePage</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetsRequest{}
client.Datasets.ListDatasets(
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

**includeArchived:** `*bool` 
    
</dd>
</dl>

<dl>
<dd>

**query:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` 
    
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

<details><summary><code>client.Datasets.CreateDataset(request) -> *chroniclego.TaskSuite</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateTaskSuitePayload{
    Name: "name",
}
client.Datasets.CreateDataset(
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

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**name:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**purpose:** `*chroniclego.CreateTaskSuitePayloadPurpose` — Intended use of a dataset — drives the colored badge on the picker and lets apps route additions to the right backend (eval suite, training set, replay corpus, manual review queue).
    
</dd>
</dl>

<dl>
<dd>

**tags:** `[]string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.CreateDatasetWithTrace(request) -> *chroniclego.CreateTaskSuiteWithTraceResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateTaskSuiteWithTraceRequest{
    Dataset: &chroniclego.CreateTaskSuiteWithTraceRequestDataset{
        Name: "name",
    },
    Trace: &chroniclego.CreateTaskSuiteWithTraceRequestTrace{
        TraceID: "traceId",
    },
}
client.Datasets.CreateDatasetWithTrace(
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

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**dataset:** `*chroniclego.CreateTaskSuiteWithTraceRequestDataset` 
    
</dd>
</dl>

<dl>
<dd>

**trace:** `*chroniclego.CreateTaskSuiteWithTraceRequestTrace` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.GetDataset(DatasetID) -> *chroniclego.TaskSuiteDetail</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetDatasetRequest{
    DatasetID: "dataset_id",
}
client.Datasets.GetDataset(
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

**datasetID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ArchiveDataset(DatasetID) -> error</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ArchiveDatasetRequest{
    DatasetID: "dataset_id",
}
client.Datasets.ArchiveDataset(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**cascade:** `*bool` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.UpdateDataset(DatasetID, request) -> *chroniclego.TaskSuite</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.TaskSuitePatch{
    DatasetID: "dataset_id",
}
client.Datasets.UpdateDataset(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**name:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**purpose:** `*chroniclego.TaskSuitePatchPurpose` — Intended use of a dataset — drives the colored badge on the picker and lets apps route additions to the right backend (eval suite, training set, replay corpus, manual review queue).
    
</dd>
</dl>

<dl>
<dd>

**tags:** `[]string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.GetDatasetSnapshot(DatasetID) -> *chroniclego.TaskSuiteSnapshot</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetDatasetSnapshotRequest{
    DatasetID: "dataset_id",
}
client.Datasets.GetDatasetSnapshot(
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

**datasetID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ListDatasetTraces(DatasetID) -> *chroniclego.TaskPage</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetTracesRequest{
    DatasetID: "dataset_id",
}
client.Datasets.ListDatasetTraces(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` 
    
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

<details><summary><code>client.Datasets.AddTraceToDataset(DatasetID, request) -> *chroniclego.AddTaskFromTraceResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.AddTaskFromTraceRequest{
    DatasetID: "dataset_id",
    TraceID: "traceId",
}
client.Datasets.AddTraceToDataset(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**eventIDs:** `[]string` — Accepted for compatibility but never trusted as the authoritative capture. The service re-reads the canonical store by subject.
    
</dd>
</dl>

<dl>
<dd>

**addTaskFromTraceRequestIdempotencyKey:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**notes:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**split:** `*chroniclego.AddTaskFromTraceRequestSplit` — Train / validation / test split assignment.
    
</dd>
</dl>

<dl>
<dd>

**task:** `*chroniclego.AddTaskFromTraceRequestTask` — Optional task fields. Anything left unset is derived from the captured trace (title from the label, instruction from the first message, expected outcome from the events after the cutoff).
    
</dd>
</dl>

<dl>
<dd>

**traceID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**traceSynthesized:** `*bool` 
    
</dd>
</dl>

<dl>
<dd>

**verifiers:** `[]*chroniclego.AddTaskFromTraceRequestVerifiersItem` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.UpdateDatasetTraces(DatasetID, request) -> error</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.UpdateTracesRequest{
    DatasetID: "dataset_id",
    Patch: &chroniclego.UpdateTracesRequestPatch{},
    TraceIDs: []string{
        "traceIds",
    },
}
client.Datasets.UpdateDatasetTraces(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**patch:** `*chroniclego.UpdateTracesRequestPatch` — Patch to apply to one or more memberships. Nullable annotations preserve the same three states as [`PatchField`]: explicit JSON `null` clears while omission is a no-op.
    
</dd>
</dl>

<dl>
<dd>

**traceIDs:** `[]string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.RemoveTraceFromDataset(DatasetID, MembershipID) -> error</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.RemoveTraceFromDatasetRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
}
client.Datasets.RemoveTraceFromDataset(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**reason:** `*string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.RefreshDatasetTrace(DatasetID, MembershipID, request) -> *chroniclego.TaskMembership</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.RefreshDatasetTraceRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
    Body: &chroniclego.RefreshMembershipRequest{},
}
client.Datasets.RefreshDatasetTrace(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**request:** `*chroniclego.RefreshMembershipRequest` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ListDatasetTraceEvents(DatasetID, MembershipID) -> *chroniclego.TaskEventPage</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetTraceEventsRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
}
client.Datasets.ListDatasetTraceEvents(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` 
    
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

<details><summary><code>client.Datasets.ListTraceDatasetMemberships(TraceID) -> []*chroniclego.TaskMembership</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListTraceDatasetMembershipsRequest{
    TraceID: "trace_id",
}
client.Datasets.ListTraceDatasetMemberships(
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

**traceID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ListDatasetTasks(DatasetID) -> *chroniclego.TaskPage</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetTasksRequest{
    DatasetID: "dataset_id",
}
client.Datasets.ListDatasetTasks(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` 
    
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

<details><summary><code>client.Datasets.CreateDatasetTask(DatasetID, request) -> *chroniclego.Task</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateDatasetTaskRequest{
    DatasetID: "dataset_id",
    Body: map[string]any{
        "key": "value",
    },
}
client.Datasets.CreateDatasetTask(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**request:** `chroniclego.CreateTaskRequest` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.GetDatasetTask(DatasetID, MembershipID) -> *chroniclego.Task</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetDatasetTaskRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
}
client.Datasets.GetDatasetTask(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.DeleteDatasetTask(DatasetID, MembershipID) -> error</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.DeleteDatasetTaskRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
}
client.Datasets.DeleteDatasetTask(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**reason:** `*string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.UpdateDatasetTask(DatasetID, MembershipID, request) -> *chroniclego.Task</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.UpdateDatasetTaskRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
    Body: map[string]any{
        "key": "value",
    },
}
client.Datasets.UpdateDatasetTask(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**request:** `chroniclego.UpdateTaskRequest` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.SetDatasetTaskVerifiers(DatasetID, MembershipID, request) -> *chroniclego.Task</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.SetTaskVerifiersRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
    Verifiers: []*chroniclego.SetTaskVerifiersRequestVerifiersItem{
        &chroniclego.SetTaskVerifiersRequestVerifiersItem{
            ScorerID: "scorerId",
        },
    },
}
client.Datasets.SetDatasetTaskVerifiers(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**verifiers:** `[]*chroniclego.SetTaskVerifiersRequestVerifiersItem` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ListDatasetTaskEvents(DatasetID, MembershipID) -> *chroniclego.TaskEventPage</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetTaskEventsRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
}
client.Datasets.ListDatasetTaskEvents(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` 
    
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

<details><summary><code>client.Datasets.RefreshDatasetTask(DatasetID, MembershipID, request) -> *chroniclego.TaskMembership</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.RefreshDatasetTaskRequest{
    DatasetID: "dataset_id",
    MembershipID: "membership_id",
    Body: &chroniclego.RefreshMembershipRequest{},
}
client.Datasets.RefreshDatasetTask(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**membershipID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**request:** `*chroniclego.RefreshMembershipRequest` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ListDatasetClusters(DatasetID) -> []*chroniclego.DatasetCluster</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetClustersRequest{
    DatasetID: "dataset_id",
}
client.Datasets.ListDatasetClusters(
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

**datasetID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.CreateDatasetCluster(DatasetID, request) -> *chroniclego.DatasetCluster</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateClusterRequest{
    DatasetID: "dataset_id",
    Color: "color",
    Label: "label",
}
client.Datasets.CreateDatasetCluster(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**color:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**createClusterRequestIdempotencyKey:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**label:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**similarityCenter:** `[]float64` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.DeleteDatasetCluster(DatasetID, ClusterID) -> error</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.DeleteDatasetClusterRequest{
    DatasetID: "dataset_id",
    ClusterID: "cluster_id",
}
client.Datasets.DeleteDatasetCluster(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**clusterID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.UpdateDatasetCluster(DatasetID, ClusterID, request) -> *chroniclego.DatasetCluster</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.UpdateClusterRequest{
    DatasetID: "dataset_id",
    ClusterID: "cluster_id",
}
client.Datasets.UpdateDatasetCluster(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**clusterID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**color:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**label:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**similarityCenter:** `[]float64` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ListDatasetSavedViews(DatasetID) -> []*chroniclego.DatasetSavedView</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetSavedViewsRequest{
    DatasetID: "dataset_id",
}
client.Datasets.ListDatasetSavedViews(
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

**datasetID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.CreateDatasetSavedView(DatasetID, request) -> *chroniclego.DatasetSavedView</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateSavedViewRequest{
    DatasetID: "dataset_id",
    Name: "name",
    Scope: chroniclego.CreateSavedViewRequestScopePersonal,
    State: &chroniclego.CreateSavedViewRequestState{},
}
client.Datasets.CreateDatasetSavedView(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**createSavedViewRequestIdempotencyKey:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**name:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**schemaVersion:** `*int` 
    
</dd>
</dl>

<dl>
<dd>

**scope:** `*chroniclego.CreateSavedViewRequestScope` 
    
</dd>
</dl>

<dl>
<dd>

**shortcut:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**state:** `*chroniclego.CreateSavedViewRequestState` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.DeleteDatasetSavedView(DatasetID, ViewID) -> error</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.DeleteDatasetSavedViewRequest{
    DatasetID: "dataset_id",
    ViewID: "view_id",
}
client.Datasets.DeleteDatasetSavedView(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**viewID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.UpdateDatasetSavedView(DatasetID, ViewID, request) -> *chroniclego.DatasetSavedView</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.DatasetSavedViewPatch{
    DatasetID: "dataset_id",
    ViewID: "view_id",
}
client.Datasets.UpdateDatasetSavedView(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**viewID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**createdBy:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**name:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**scope:** `*chroniclego.DatasetSavedViewPatchScope` 
    
</dd>
</dl>

<dl>
<dd>

**shortcut:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**state:** `*chroniclego.DatasetSavedViewPatchState` 
    
</dd>
</dl>

<dl>
<dd>

**updatedAt:** `*time.Time` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ListDatasetVersions(DatasetID) -> []*chroniclego.TaskSuiteVersion</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetVersionsRequest{
    DatasetID: "dataset_id",
}
client.Datasets.ListDatasetVersions(
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

**datasetID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.PublishDatasetVersion(DatasetID, request) -> *chroniclego.TaskSuiteVersion</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.PublishVersionRequest{
    DatasetID: "dataset_id",
}
client.Datasets.PublishDatasetVersion(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**publishVersionRequestIdempotencyKey:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**label:** `*string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.GetDatasetVersion(DatasetID, VersionID) -> *chroniclego.TaskSuiteSnapshot</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetDatasetVersionRequest{
    DatasetID: "dataset_id",
    VersionID: "version_id",
}
client.Datasets.GetDatasetVersion(
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

**datasetID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**versionID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Datasets.ListDatasetEvaluationRuns(DatasetID) -> []*chroniclego.TaskSuiteEvalRun</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListDatasetEvaluationRunsRequest{
    DatasetID: "dataset_id",
}
client.Datasets.ListDatasetEvaluationRuns(
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

**datasetID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## environments
<details><summary><code>client.Environments.ListEnvironments() -> *chroniclego.ListEnvironmentsResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
client.Environments.ListEnvironments(
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

<details><summary><code>client.Environments.CreateEnvironment(request) -> *chroniclego.EnvironmentResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateEnvironmentRequest{
    Slug: "slug",
    Label: "label",
}
client.Environments.CreateEnvironment(
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

**slug:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**label:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**description:** `*string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Environments.GetEnvironment(EnvironmentID) -> *chroniclego.EnvironmentResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetEnvironmentRequest{
    EnvironmentID: "environment_id",
}
client.Environments.GetEnvironment(
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

**environmentID:** `string` — Environment ID or slug.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Environments.ListEnvironmentVersions(EnvironmentID) -> []*chroniclego.EnvironmentVersionRecord</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListEnvironmentVersionsRequest{
    EnvironmentID: "environment_id",
}
client.Environments.ListEnvironmentVersions(
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

**environmentID:** `string` — Environment ID or slug.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Environments.CreateEnvironmentVersion(EnvironmentID, request) -> *chroniclego.EnvironmentVersionResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CreateEnvironmentVersionRequest{
    EnvironmentID: "environment_id",
    Version: "version",
}
client.Environments.CreateEnvironmentVersion(
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

**environmentID:** `string` — Environment ID or slug.
    
</dd>
</dl>

<dl>
<dd>

**version:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**spec:** `*chroniclego.EnvironmentSpec` 
    
</dd>
</dl>

<dl>
<dd>

**status:** `*chroniclego.EnvironmentVersionStatus` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Environments.GetEnvironmentVersion(EnvironmentID, VersionSelector) -> *chroniclego.EnvironmentVersionResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetEnvironmentVersionRequest{
    EnvironmentID: "environment_id",
    VersionSelector: "version_selector",
}
client.Environments.GetEnvironmentVersion(
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

**environmentID:** `string` — Environment ID or slug.
    
</dd>
</dl>

<dl>
<dd>

**versionSelector:** `string` — Environment-version ID or version label.
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Environments.CompileEnvironmentVersion(EnvironmentID, VersionSelector, request) -> *chroniclego.CompileEnvironmentResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CompileEnvironmentRequest{
    EnvironmentID: "environment_id",
    VersionSelector: "version_selector",
    DatasetSnapshotID: "datasetSnapshotId",
    ScenarioID: "scenarioId",
}
client.Environments.CompileEnvironmentVersion(
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

**environmentID:** `string` — Environment ID or slug.
    
</dd>
</dl>

<dl>
<dd>

**versionSelector:** `string` — Environment-version ID or version label.
    
</dd>
</dl>

<dl>
<dd>

**datasetSnapshotID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**scenarioID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## backtests
<details><summary><code>client.Backtests.GetBacktestsAvailability() -> *chroniclego.BacktestsAvailability</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
client.Backtests.GetBacktestsAvailability(
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

<details><summary><code>client.Backtests.ListBacktestJobs() -> *chroniclego.ListBacktestJobsResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListBacktestJobsRequest{}
client.Backtests.ListBacktestJobs(
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

**mode:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**status:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` 
    
</dd>
</dl>

<dl>
<dd>

**offset:** `*int` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Backtests.CreateBacktestJob(request) -> *chroniclego.CreateBacktestJobResponse</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

Returns 202 after the durable job and its trials have been admitted.
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
request := &chroniclego.CreateBacktestJobRequest{
    Name: "name",
    Recipe: &chroniclego.CreateBacktestJobRequestRecipe{
        Agents: []*chroniclego.CreateBacktestJobRequestRecipeAgentsItem{
            &chroniclego.CreateBacktestJobRequestRecipeAgentsItem{
                Hue: "hue",
                ID: "id",
                Label: "label",
                Notes: "notes",
            },
        },
        Data: &chroniclego.CreateBacktestJobRequestRecipeData{
            Kind: chroniclego.CreateBacktestJobRequestRecipeDataKindComposed,
            Scenarios: []*chroniclego.CreateBacktestJobRequestRecipeDataScenariosItem{
                &chroniclego.CreateBacktestJobRequestRecipeDataScenariosItem{
                    Count: 1,
                    ID: "id",
                    Kind: chroniclego.CreateBacktestJobRequestRecipeDataScenariosItemKindAdversarial,
                    Label: "label",
                },
            },
            Sources: []*chroniclego.CreateBacktestJobRequestRecipeDataSourcesItem{
                &chroniclego.CreateBacktestJobRequestRecipeDataSourcesItem{
                    Count: 1,
                    ID: "id",
                    Kind: chroniclego.CreateBacktestJobRequestRecipeDataSourcesItemKindProd,
                    Label: "label",
                },
            },
        },
        Graders: []*chroniclego.CreateBacktestJobRequestRecipeGradersItem{
            &chroniclego.CreateBacktestJobRequestRecipeGradersItem{
                ID: "id",
                Kind: chroniclego.CreateBacktestJobRequestRecipeGradersItemKindRubric,
                Label: "label",
                Source: chroniclego.CreateBacktestJobRequestRecipeGradersItemSourceProposed,
                Weight: chroniclego.CreateBacktestJobRequestRecipeGradersItemWeightLow,
            },
        },
        Mode: chroniclego.CreateBacktestJobRequestRecipeModeReplay,
        Name: "name",
    },
}
client.Backtests.CreateBacktestJob(
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

**idempotencyKey:** `*string` — Optional caller-generated key for safely retrying a mutation.
    
</dd>
</dl>

<dl>
<dd>

**cases:** `[]*chroniclego.CreateBacktestJobRequestCasesItem` 
    
</dd>
</dl>

<dl>
<dd>

**evaluatorProfileID:** `*string` 
    
</dd>
</dl>

<dl>
<dd>

**nConcurrent:** `*int` 
    
</dd>
</dl>

<dl>
<dd>

**name:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**recipe:** `*chroniclego.CreateBacktestJobRequestRecipe` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Backtests.GetBacktestJob(JobID) -> *chroniclego.BacktestJobDetailResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetBacktestJobRequest{
    JobID: "job_id",
}
client.Backtests.GetBacktestJob(
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

**jobID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Backtests.ListBacktestJobTrials(JobID) -> *chroniclego.ListBacktestJobTrialsResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.ListBacktestJobTrialsRequest{
    JobID: "job_id",
}
client.Backtests.ListBacktestJobTrials(
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

**jobID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**limit:** `*int` 
    
</dd>
</dl>

<dl>
<dd>

**offset:** `*int` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Backtests.GetBacktestTrial(JobID, TrialID) -> *chroniclego.BacktestTrialDetailResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.GetBacktestTrialRequest{
    JobID: "job_id",
    TrialID: "trial_id",
}
client.Backtests.GetBacktestTrial(
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

**jobID:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**trialID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Backtests.CancelBacktestJob(JobID) -> *chroniclego.CancelBacktestJobResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.CancelBacktestJobRequest{
    JobID: "job_id",
}
client.Backtests.CancelBacktestJob(
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

**jobID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Backtests.StreamBacktestJobEvents(JobID) -> chroniclego.TrialEvent</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.StreamBacktestJobEventsRequest{
    JobID: "job_id",
}
client.Backtests.StreamBacktestJobEvents(
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

**jobID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

## credentials
<details><summary><code>client.Credentials.ListSdkKeys() -> *chroniclego.SdkKeyListResponse</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
client.Credentials.ListSdkKeys(
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

<details><summary><code>client.Credentials.CreateSdkKey(request) -> *chroniclego.CreatedSdkKey</code></summary>
<dl>
<dd>

#### 📝 Description

<dl>
<dd>

<dl>
<dd>

The bearer secret is returned once and is not stored in plaintext.
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
request := &chroniclego.CreateSdkKeyRequest{
    Name: "name",
    Scopes: []chroniclego.CreateSdkKeyRequestScopesItem{
        chroniclego.CreateSdkKeyRequestScopesItemTracesWrite,
    },
}
client.Credentials.CreateSdkKey(
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

**name:** `string` 
    
</dd>
</dl>

<dl>
<dd>

**scopes:** `[]*chroniclego.CreateSdkKeyRequestScopesItem` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

<details><summary><code>client.Credentials.RevokeSdkKey(KeyID) -> error</code></summary>
<dl>
<dd>

#### 🔌 Usage

<dl>
<dd>

<dl>
<dd>

```go
request := &chroniclego.RevokeSdkKeyRequest{
    KeyID: "key_id",
}
client.Credentials.RevokeSdkKey(
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

**keyID:** `string` 
    
</dd>
</dl>
</dd>
</dl>


</dd>
</dl>
</details>

