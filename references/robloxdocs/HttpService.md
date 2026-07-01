# HttpService
> Roblox Luau reference for external API communication, serialization, security, and request control.
> Use this file as the single HttpService guide for AI reasoning.

## 1. Purpose

HttpService is the bridge between Roblox and the outside world.
It is used for server-side communication with web APIs, external tools, and remote data sources.

Use it when you want:
- external database or analytics integration
- dynamic content fetch
- webhook delivery
- unique request identifiers

Do not use it:
- on the client
- for high-frequency streaming
- to expose secrets from replicated code

## 2. Core Principles

1. HttpService is server-side only for sensitive logic.
2. All requests are asynchronous in practice.
3. JSON is the default data format.
4. External APIs fail sometimes; design for failure.
5. Rate limits are real on both sides.
6. External data is untrusted input.
7. HTTP must be explicitly enabled in game settings.

## 3. API Surface

### 3.1 RequestAsync

`HttpService:RequestAsync(requestTable)`

- sends an HTTP request
- accepts `Url`, `Method`, `Headers`, and `Body`
- returns a response table with success and status information

### 3.2 JSONEncode / JSONDecode

- `HttpService:JSONEncode(value)` converts Lua data to JSON
- `HttpService:JSONDecode(jsonString)` converts JSON to Lua data
- invalid shapes will error

### 3.3 GenerateGUID

`HttpService:GenerateGUID(shouldAddBrackets)`

- returns a unique identifier
- useful for request tracking and external IDs

## 4. Golden Pattern

Never raw-call `RequestAsync` in gameplay flow.
Wrap it in one reusable request function.

```lua
local HttpService = game:GetService("HttpService")

local function Request(url: string, method: string, data: any?)
    local ok, response = pcall(function()
        return HttpService:RequestAsync({
            Url = url,
            Method = method,
            Headers = {
                ["Content-Type"] = "application/json",
            },
            Body = data and HttpService:JSONEncode(data) or nil,
        })
    end)

    if not ok then
        warn("Http request failed:", response)
        return nil
    end

    if not response.Success then
        warn("API error:", response.StatusCode, response.StatusMessage)
        return nil
    end

    return HttpService:JSONDecode(response.Body)
end
```

## 5. Security Rules

- keep secrets on the server only
- never put API keys in replicated code
- use a backend or proxy when secrets must stay hidden
- validate decoded payloads before use
- reject malformed or unexpected data
- use `https://` endpoints only

## 6. Rate Limiting and Performance

- do not fire requests inside tight loops
- use debounce for repeated player-triggered requests
- batch updates when possible
- add exponential backoff on retry
- keep requests inside `task.spawn` or `task.defer` when they can yield
- do not let requests block the main gameplay flow

## 7. Error Handling Matrix

| Status | Meaning | Action |
|---|---|---|
| 200-299 | success | process data |
| 400 | bad request | check payload and headers |
| 401/403 | unauthorized | check key and permissions |
| 404 | not found | verify endpoint |
| 429 | too many requests | back off |
| 500+ | server error | log and retry later |

## 8. Serialization Rules

JSON is not unlimited.

Allowed:
- strings
- numbers
- booleans
- nested tables

Not allowed:
- Instances
- functions
- threads
- userdata

Rules:
- convert Instances into IDs or plain data
- avoid cyclic tables
- avoid mixed table shapes when possible
- validate structure after decode

## 9. Registry Pattern

For larger systems, use one API manager.

Rules:
- centralize base URLs
- centralize shared headers
- centralize retry logic
- centralize logging
- keep gameplay logic separate from transport logic

## 10. Request Flow

1. validate input
2. serialize data
3. send request
4. inspect response
5. decode response
6. validate decoded payload
7. hand result to gameplay code

## 11. Common Failures

### 11.1 Blocking the main thread

Problem: request runs inside a sensitive synchronous flow.
Fix: move it into `task.spawn` or `task.defer`.

### 11.2 Missing pcall

Problem: outage crashes the script.
Fix: wrap every request.

### 11.3 Exposed keys

Problem: secrets are stored where the client can see them.
Fix: keep them server-only.

### 11.4 Malformed JSON

Problem: payload is not encoded correctly.
Fix: encode before send and validate after decode.

### 11.5 Retry spam

Problem: request retries happen too fast.
Fix: add cooldown and backoff.

## 12. Professional Behavior Rules for AI

- check whether the request runs on the server
- wrap risky requests in `pcall`
- validate headers and payload shape
- prefer asynchronous execution
- warn when client-side sensitive requests are attempted
- default to defensive behavior

## 13. Minimal Checklist

- HTTP requests are enabled
- API keys are server-only
- every request uses `pcall`
- payload is validated
- rate limiting exists
- retries are controlled
- request flow is asynchronous
- errors are logged with context

## 14. Final Rule

HttpService is a privilege, not a guarantee.
Design it to fail gracefully so gameplay stays intact when the network does not.

