# AI slop

Slop is code that looks finished and is not: volume without value, ceremony
without need, or a plausible shape with nothing behind it. It appears in
AI-assisted work more often than in hand-written work, but the origin is not
the point and you cannot prove it.

## Rules before you flag anything

**Never name the author or the tool.** Do not write "AI-generated", "looks
like ChatGPT", "LLM boilerplate", or "the model added". Write the defect:
"three layers re-validate the same input", "this helper wraps one stdlib call",
"this returns hardcoded data". The defect is reviewable. The accusation is not.

**Match the repo, not a standard.** If the repo is verbose, verbose is correct.
If every module has a banner comment, the new banner is not a finding. Read two
neighbouring files before you call something excessive.

**Most slop is a Nit.** Promote it only when it earns the level:

| Level | When |
|---|---|
| Blocker | A stub or fake presented as working: hardcoded return, `pass  # implement later`, sample data, an API or flag that does not exist |
| Warning | It hides a real defect, or it adds a structure the team must now maintain forever |
| Nit | It is only noise |

**One line, not one file.** Point at a line number. Never write "this file is
slop".

**Good engineering is not slop.** Type hints, guard clauses, docstrings on
public API, defensive checks at a trust boundary, and thorough tests are
correct. The question is never "is there a lot of it" — it is "does the reader
pay more than they get".

## Signals

### Fake completeness

The highest-value catch in this whole reference.

```python
def calculate_risk(user):
    return 0.5          # plausible, arbitrary, wrong
```
```typescript
async function fetchMetrics() {
  return { users: 1200, revenue: 45000 };   // sample data in a real path
}
```
```go
func Migrate() error {
    // TODO: implement
    return nil          // reports success, does nothing
}
```

Also: a function whose body ignores its own parameters, a test that asserts
`true`, an error branch that returns the happy-path value.

### Invented APIs

A method, flag, config key, or import that does not exist, spelled the way it
plausibly should be. Verify anything you do not recognise:
`rg -n "methodName"`, or check the dependency's source. Wrong API is a Blocker.

### Ceremony with one user

```python
class ConfigFactory:              # one implementation, one call site
    def create(self) -> Config: ...
```

- An interface or protocol with a single implementation and no test double.
- A helper called once that wraps one standard-library call.
- A parameter, option, or `**kwargs` passthrough that no caller sets.
- A constants file holding two values used in one place.
- A wrapper class over a dict.

Fine when the repo already works this way, or when the second implementation is
in the same diff.

### Redundant defence

```python
def handler(payload: dict):
    if payload is None: ...           # the caller already checked
    if not isinstance(payload, dict): ...
    try:
        value = payload["id"]
    except KeyError:
        return None                   # caller cannot tell this from a real miss
```

Re-validating at every layer is not safety; it spreads the question "who owns
this check" across the codebase. Defence at the trust boundary is correct.
Defence three frames in is noise.

### Symmetric no-ops

```javascript
if (ok) { return true; } else { return false; }
try { return doIt(); } catch (e) { throw e; }
const result = value;
return result;
```

### Comment noise

Section banners, a comment above every line, comments that restate the code,
comments that address the requester ("As requested", "Changed from 60 because
you mentioned"), hedges ("this might need optimisation"), and docstrings that
repeat the signature. `document-changes` handles these in depth; in a review,
one Nit row covering the file is enough.

### Prose tells

In new docs, comments, or error messages: *comprehensive, robust, seamless,
leverage, delve, ensure, it is worth noting, in today's landscape*, a bulleted
list where one sentence fits, and a README with Contributing and License
sections for an internal script. Flag it once, as a Nit, when it is new prose
this change introduced.

### Test theatre

```python
def test_process_works():
    mock.process.return_value = 42
    assert service.run() == 42          # asserts the mock, not the code
```

- Tests that assert a mock was called and nothing else.
- Tests that restate the implementation line for line.
- One test per getter, none for the branch that matters.
- Names like `test_function_works`, `test_happy_path`, `test_edge_cases`.

Coverage that only exercises the path the author already knew works is a
Warning when the change is risky, a Nit otherwise.

### Suppression instead of a fix

A `# type: ignore`, `# noqa`, `@SuppressWarnings`, `eslint-disable-next-line`,
or `//nolint` added in this diff with no narrowing code and no reason. Ask what
the tool was complaining about. Broad, unexplained suppression is a Warning.

### Reinvention

Logic the repo already has, three directories away. Search before you claim it,
and give the existing symbol in the fix column:
`rg -n "def parse_duration|parseDuration"`.

## Writing the finding

| Weak | Strong |
|---|---|
| Over-engineered, looks AI-generated | `config/factory.py:14` — interface with one implementation and no test double |
| Too many comments | `api/routes.py:1-40` — comments restate each line; delete |
| Suspicious code | `risk.py:22` — returns a hardcoded `0.5`; `user` is unused |
