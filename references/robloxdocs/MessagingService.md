# MessagingService
> Roblox Luau reference for inter-server communication, topic broadcasting, and cross-server coordination.
> Use this file as the single MessagingService guide for AI reasoning.

## 1. Purpose

MessagingService is a Roblox engine service for live server-to-server communication.
It is used when multiple servers need to share short, transient information without persistence.

Use it when you want:
- global announcements
- cross-server coordination
- matchmaking signals
- moderation or relay messages

Do not use it as:
- a datastore
- a reliable queue
- a client communication channel

## 2. Core Principles

1. Messaging is asynchronous and transient.
2. Messages are not saved.
3. Delivery is not guaranteed.
4. Only servers participate.
5. Payloads must be serialized.
6. Every subscription needs a cleanup path.
7. Topic names should come from a registry.
8. Cross-server state is eventually consistent, not atomic.

## 3. API Surface

### 3.1 PublishAsync

`MessagingService:PublishAsync(topic, message)`

- broadcasts a message to all subscribed servers
- yields the thread
- should not be spammed

### 3.2 SubscribeAsync

`MessagingService:SubscribeAsync(topic, callback)`

- registers a listener for a topic
- returns a connection-like object
- callback receives a table with message data and timestamp

## 4. Limits and Quotas

MessagingService has strict limits.

- payloads are small
- topic names should stay short
- message rate must be controlled

Rules:
- batch when possible
- compress payload shape
- never publish every frame
- never rely on unlimited throughput

## 5. Payload Design

Use a small structured payload.

Rules:
- prefer tables with clear keys
- keep schemas consistent
- validate on receipt
- treat incoming data as untrusted

Example shape:

```lua
{
    Action = "Announcement",
    Payload = {
        Text = "Server restart soon",
    }
}
```

## 6. Lifecycle Pattern

The correct flow is:
1. define a topic registry
2. subscribe at server startup
3. validate payloads before use
4. clean up connections on shutdown

### Standard pattern

```lua
local MessagingService = game:GetService("MessagingService")

local TOPIC = "GlobalAnnouncement"

local function onMessage(message)
    local data = message.Data
    if type(data) ~= "table" then
        return
    end
end

local ok, connection = pcall(function()
    return MessagingService:SubscribeAsync(TOPIC, onMessage)
end)

if not ok then
    warn("Subscribe failed:", connection)
end
```

## 7. Manager Pattern

For production code, use a wrapper or manager.

Rules:
- centralize topic names
- centralize publish logic
- centralize logging
- centralize cleanup
- keep game logic separate from transport logic

This prevents string littering and scattered subscription handling.

## 8. Error Handling

MessagingService can fail.

Rules:
- wrap publish and subscribe calls in `pcall`
- log failures without crashing gameplay
- expect delay or message loss
- do not assume arrival order is stable

## 9. Security Rules

MessagingService is server-to-server only.

Rules:
- never send secrets
- never trust sender identity without validation
- validate schema and permissions
- rate-limit any expensive action triggered by a message
- assume any server can publish to the topic

## 10. Performance Rules

- do not publish in frame loops
- use short payloads
- offload heavy work from the callback
- use `task.spawn` when the handler may take time
- reserve MessagingService for necessary cross-server sync

## 11. Common Failures

### 11.1 Reliable delivery assumption

Problem: code expects every message to arrive.
Fix: design for missed messages.

### 11.2 Payload bloat

Problem: payload is too large.
Fix: reduce shape or send an ID reference instead.

### 11.3 Subscription leak

Problem: subscriptions remain active when no longer needed.
Fix: store the connection and clean it up.

### 11.4 Missing pcall

Problem: publish or subscribe error breaks the flow.
Fix: wrap in `pcall`.

## 12. Professional Behavior Rules for AI

- prioritize robustness over speed
- validate every payload
- centralize topic names
- assume network failure is normal
- keep receivers decoupled from sender identity
- keep the message contract small and explicit

## 13. Mental Model

Think in layers:
- topic: the channel
- wrapper: the safety layer
- payload: the contract

If the contract is vague, the system will drift and break.

## 14. Minimal Checklist

- topics are centralized
- payload size is small
- calls are wrapped in `pcall`
- incoming data is validated
- subscriptions are cleaned up
- publish rate is controlled

## 15. Final Rule

MessagingService is for coordination, not data integrity.
Keep messages brief, validated, and disposable.

