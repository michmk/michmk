# Silent failures

A silent failure is code that goes wrong and reports success. Nobody sees an
error, no alert fires, and the wrong result flows on. These are the most
valuable findings in a review, because tests rarely catch them and users report
them as "the data is wrong" months later.

## The test

For each error path in the diff, ask three questions:

1. **Who learns that it failed?** Trace to the caller. If the caller receives a
   normal-looking value, it is silent.
2. **What does the system do next?** If it continues as though the work
   succeeded, it is silent.
3. **Would you find it at 3am?** No log, no metric, no exception, no non-zero
   exit — then no.

If the answer to all three is bad, it is a **Blocker**. If it is observable but
easy to miss (logged and ignored, counted but not alerted), it is a **Warning**.

## Patterns

### Swallowed errors

```python
try:
    sync()
except Exception:
    pass
```
```javascript
fetchUser().catch(() => {});
promise.then(ok);            // no rejection handler
```
```go
_ = json.Unmarshal(b, &cfg)  // parse failed, cfg is zero
f.Close()                    // write buffer never flushed
```
```java
try { commit(); } catch (Exception e) { }
```
```ruby
value = JSON.parse(body) rescue nil
```
```c
fwrite(buf, 1, n, f);        // return value ignored, short write invisible
```

Look for: empty catch, `pass`, `return null`, `continue`, blanket `except
Exception` / `catch (Throwable)`, discarded error variables, `.catch(noop)`.

### Logging is not handling

```python
except ValueError as e:
    log.error(e)
    # falls through and uses the stale value
```

The log satisfies the reviewer and nothing else. Ask what the code does *after*
the log line. If it proceeds as if nothing happened, that is the finding.

### Defaults that hide a miss

```python
port = config.get("port", 8080)      # typo in the key is now invisible
```
```javascript
const limit = opts.limit ?? 100;
const name = user.name || "anonymous";   // "" and 0 fall through too
```
```rust
let n = s.parse::<u32>().unwrap_or_default();   // bad input becomes 0
```

A default is correct when absence is a legitimate state. It is a bug when
absence means the caller made a mistake, or when the fallback is
indistinguishable from a real value.

### Type coercion

```javascript
parseInt("12abc")   // 12
parseInt(undefined) // NaN, then NaN spreads silently
Number("")          // 0
```
```python
int(3.99)           # 3
bool("false")       # True
```

Also: integer overflow that wraps, float equality, precision lost in a
narrowing cast, string truncation at a column limit.

### Loops that skip

```python
for row in rows:
    try:
        process(row)
    except Exception:
        continue          # how many were skipped? nobody knows
```

Batch work needs a count of failures and a decision rule. Flag any loop that
drops items with no tally and no threshold.

### Non-atomic multi-step work

Two writes, a crash between them, and the system is half-updated with no error:
`INCR` then `EXPIRE`, insert then index, write file then rename, charge then
record. Ask what the state looks like if the process dies between the steps.

### Async work nobody waits for

```javascript
async function save() { ... }
save();                       // floating promise, rejection unhandled
```
```python
asyncio.create_task(sync())   # no reference, may be garbage collected
```
```go
go doWork()                   // error has nowhere to go
```
```csharp
_ = SendAsync();              // fire and forget
```

### Missing timeout or missing cancellation

No timeout turns a failure into a hang, which reads as "slow" rather than
"broken". A timeout that returns an empty result instead of an error is worse.

### Validation that does not gate

```python
assert user.is_valid()        # stripped by python -O
validate(payload)             # returns errors; return value discarded
if not ok:
    log.warning("invalid")    # and then continues anyway
```

### Query results nobody checks

`UPDATE` that matched zero rows, `DELETE` that deleted nothing, an upsert that
hit a conflict clause, a deserialiser that dropped unknown fields, a migration
that ran against the wrong schema.

### Shell

```bash
set -u only                # no -e, no -o pipefail
curl "$URL" | tar xz       # curl fails, tar succeeds on empty input
cd "$DIR"; rm -rf ./*      # cd failed, and now you are in $HOME
```

Flag a script that has no `set -euo pipefail` and does destructive or
sequential work.

### Config and flags

A misspelled environment variable that falls back to a default, a feature flag
that no-ops when unset, a code path guarded by a condition that is never true.

## Legitimately silent

Do not flag these unless the surrounding intent says otherwise:

| Pattern | Why it is fine |
|---|---|
| Cache read error treated as a miss | The fallback recomputes the real value |
| Best-effort cleanup in `defer` / `finally` | The operation already succeeded |
| Optional telemetry or metrics send | Not on the critical path |
| Retry loop that logs each attempt and raises at the end | The failure escapes |
| `get(key, default)` where the default is the documented value | Absence is valid |
| Ignoring "already exists" on a create | Idempotent by design |

## Writing the finding

Name the trigger and the wrong outcome, not the code shape.

Weak: `limiter/redis.go:88` — error is swallowed.
Strong: `limiter/redis.go:88` — Redis error returns `allowed=true`; an outage
removes the rate limit for every caller.
