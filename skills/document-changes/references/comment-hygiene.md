# Comment and docstring hygiene

How to decide what a comment is worth, and what a docstring must contain.

## The test

A comment earns its place if a competent reader of this codebase, who was not
present when the code was written, still needs it in six months.

That reader cannot see the chat log, the ticket you had open, the earlier
version of the file, or the question the user asked. If the comment only makes
sense to someone who saw those things, it is noise. Delete it, or convert it
into a durable fact.

## Remove: chat history residue

The most common AI leftover. The comment addresses the person who asked for the
change, not the person who will read the code.

```python
# As requested, this now uses a set instead of a list
seen = set()

# Changed from 60 because you mentioned the API was timing out
TIMEOUT = 30

# I've added error handling here as discussed
except ValueError:
    return None

# Note: I kept the original behavior for backwards compatibility
```

Every one of these describes an edit. Delete all four comment lines. The code
already is a set, already is 30, already handles the error.

The third case needs a second look. If the timeout value has a real reason,
recover it and write it durably:

```python
TIMEOUT = 30  # Upstream gateway closes idle connections at 35s.
```

If you cannot find a durable reason, write no comment. A bare `TIMEOUT = 30` is
better than a reason the reader cannot check.

## Remove: edit narration

Documentation describes state. These describe a transition, and they rot the day
someone edits the file again.

```go
// New helper to replace the old parsing logic
// Updated to handle the nil case as well
// This function was extracted from Run()
// Temporary fix until we refactor the client
// Previously this used a mutex
```

Delete them. If the "temporary fix" is real, it needs a ticket and an owner, or
it is not tracked and the comment is a lie:

```go
// FIXME(PLAT-4821): drop when the client exposes a context-aware Dial.
```

## Remove: restatement

The comment says what the next line says.

```java
// Increment the counter
counter++;

// Loop over all users
for (User u : users) {

// Return the result
return result;
```

```python
def get_name(self) -> str:
    """Get the name."""      # adds nothing over the signature
    return self._name
```

Delete the comment. For the docstring, either say something real (what the name
identifies, whether it can be empty, where it comes from) or delete it if the
repo does not require a docstring on trivial accessors.

## Remove: tutorial comments

The comment teaches the language, not the code.

```python
# A dictionary stores key-value pairs
config = {}

# List comprehensions are faster than loops
names = [u.name for u in users]
```

Delete. Assume the reader knows the language.

## Remove: hedging and apology

```cpp
// Not sure if this is the best approach, but it works
// This might need optimization later
// Hopefully this handles all the edge cases
// This is a bit hacky
```

Delete. If the concern is real, it belongs in an issue with a description, not
in a comment that no tool will ever surface.

## Remove: decoration

```python
# =====================================
#           HELPER FUNCTIONS
# =====================================

#################
# main logic
#################
```

Delete, unless the repo uses this pattern consistently. Banners hide structure
problems; they do not solve them.

## Remove: undocumented provenance

The hardest case, and the one the user asked about specifically. A comment
explains a value or a choice, and the explanation exists nowhere else — not in
the code, not in a doc, not in a ticket. It came from a conversation.

```python
retries = 3  # three was enough in testing
BATCH = 500  # this size worked well when we tried it
if user.role == "admin":  # admins are the only ones who need this per the spec discussion
```

Ask two questions:

1. **Can the reader verify it?** "Per the spec discussion" points at nothing.
   "Per RFC 7231 §6.5.1" points at something.
2. **Does anything else in the repo mention it?** Search first:
   `rg -n "retries|BATCH" --type-not lock` and `rg -ln "batch" -g '*.md'`.

If the fact is real and durable, anchor it:

```python
BATCH = 500  # Server rejects payloads over 512 items (see api/limits.md).
```

If the fact is real but lives only in the user's head, ask the user for the
source, or delete the comment and mention it in your report. If it is a
guess dressed as a reason, delete it.

## Keep and improve

These comments do work no reader can do alone:

| Kind | Example |
|---|---|
| Invariant | `// Callers must hold mu. Deadlocks if taken twice.` |
| Ordering | `# Must run before load_plugins(); plugins read this registry.` |
| Units and range | `timeout_ms: int  # milliseconds, 0 disables the timeout` |
| Magic number origin | `MAX_UDP = 65507  # 65535 minus the IPv4 and UDP headers.` |
| Workaround with a source | `// Works around golang/go#54332; remove after Go 1.26.` |
| Non-obvious algorithm | `# Two-pass: pass one sizes the buffer, pass two fills it.` |
| Safety warning | `# WARNING: overwrites the target file without a backup.` |
| Deliberate omission | `// No retry here: the caller owns the retry budget.` |

Improve these rather than delete them. Tighten the wording, add the missing
reference, correct what the change made wrong.

## Docstrings

### What every public symbol needs

- A one-line summary. It fits on one line and ends with a period.
- The parameters, with meaning and constraints — not a restatement of the types.
- The return value, with meaning — not just the type.
- The errors, exceptions, or panics a caller must handle.
- A short example when the call is not obvious from the signature.

Skip private helpers unless they are hard to follow. Never write a docstring
that adds nothing over the signature.

### Python

Follow PEP 257 and the repo's style (Google, NumPy, or Sphinx). Summary line in
the imperative or third person — match the repo. Do not repeat annotated types.

```python
def rotate_key(key_id: str, *, grace_period: timedelta = timedelta(hours=1)) -> Key:
    """Replace the key and keep the old one valid for a short time.

    Args:
        key_id: Identifier of the key to replace.
        grace_period: Time the old key stays valid. Set it to zero to
            revoke the old key immediately.

    Returns:
        The new key. The old key is not part of the result.

    Raises:
        KeyNotFoundError: The key_id does not exist.
        PermissionError: The caller cannot rotate this key.
    """
```

### Go

The comment starts with the identifier name and is a complete sentence. Use
`// Deprecated:` for deprecation.

```go
// RotateKey replaces the key and keeps the old key valid for gracePeriod.
// A zero gracePeriod revokes the old key immediately.
// It returns ErrKeyNotFound if the key does not exist.
func RotateKey(ctx context.Context, keyID string, gracePeriod time.Duration) (*Key, error) {
```

### Java

Javadoc. The first sentence is the summary and ends with a period. Use
`@param`, `@return`, `@throws`.

```java
/**
 * Replaces the key and keeps the old key valid for a short time.
 *
 * @param keyId identifier of the key to replace
 * @param gracePeriod time the old key stays valid; {@code Duration.ZERO}
 *                    revokes the old key immediately
 * @return the new key
 * @throws KeyNotFoundException if the key does not exist
 */
```

### C++

Doxygen, in the repo's marker style (`///` or `/** */`).

```cpp
/// Replaces the key and keeps the old key valid for a short time.
///
/// @param key_id Identifier of the key to replace.
/// @param grace_period Time the old key stays valid. Zero revokes the old
///     key immediately.
/// @return The new key.
/// @throws KeyNotFound If the key does not exist.
/// @note The caller owns the returned pointer.
```

### TypeScript and JavaScript

TSDoc or JSDoc. Do not restate TypeScript types in `@param` tags.

```ts
/**
 * Replaces the key and keeps the old key valid for a short time.
 *
 * @param keyId - Identifier of the key to replace.
 * @param gracePeriod - Milliseconds the old key stays valid. `0` revokes
 *   the old key immediately.
 * @throws {KeyNotFoundError} The key does not exist.
 */
```

### Docstring rot

The change may have made an existing docstring wrong. Check each one against
the current signature:

- a parameter that no longer exists, or a new one that is missing,
- a described default that no longer matches the code,
- a return description for a value the function no longer returns,
- an exception the function no longer raises, or a new one it does,
- an example that would now fail,
- a `Deprecated` note for something that is no longer deprecated.

Fix these even when the diff did not touch the docstring line.
