# README and prose docs

## Find the docs the change affects

```bash
# Docs near the changed files
fd -e md -e rst -e txt . <changed-dir>

# Docs that name a changed symbol, flag, command, or env var
rg -ln "MyClass|--my-flag|MY_ENV_VAR" -g '*.md' -g '*.rst' -g '*.adoc'

# Common doc locations
ls README* docs/ doc/ CONTRIBUTING* 2>/dev/null
```

Read every hit before you edit one. A fact often appears in the root README and
in a package README at the same time. Both must agree.

## When to update

Update an existing doc when the change touched any of these:

- a public function, class, endpoint, or CLI command the doc names,
- install, build, or test commands,
- how the reader runs the thing,
- configuration keys, environment variables, or defaults,
- required versions or dependencies,
- file and directory layout the doc describes,
- behavior the doc describes in prose, even when no name changed.

Also fix what you find broken while you are in the file: a command that no
longer exists, a renamed flag, a dead relative link, an output sample that no
longer matches.

## When to create

Create a README only when both are true:

1. The change adds a component, package, service, or tool, **and**
2. a reader must run it or import it, **and** no README covers it.

One README per component. Do not add a README to every directory. Do not create
a doc for an internal helper package that only the repo itself imports.

If you are unsure, do not create the file. Say so in your report and let the
user decide.

## Structure for a new README

Use only the sections that have content. An honest four-section README beats a
template with empty headings.

```markdown
# <name>

<One sentence: what it does and for whom.>

## Install

<The exact commands.>

## Use

<The smallest example that works, then the common options.>

## Configure

<Table: variable or key, meaning, default.>

## Develop

<How to run the tests and the build.>
```

Rules for the content:

- The first sentence says what the thing does. Not "This repository contains…".
- Every command block is copy-and-paste ready and has been checked.
- Show real output only when the reader needs it to know the run worked.
- Configuration goes in a table with a default column.
- Link to deeper docs; do not repeat them. One fact lives in one place.
- No badges you cannot verify. No "Contributing" or "Licence" section invented
  from nothing — copy the repo's existing convention or leave it out.

## Never put these in a README

| Do not write | Why |
|---|---|
| "Recent changes", "What's new", "Changelog" | The git log holds this. A README describes the present. |
| "Recently added support for X" | Describe X. Drop "recently added". |
| "This was refactored to use Y" | Say that it uses Y. |
| "TODO: document this" | Write it, or leave the section out. |
| A feature that does not exist yet | Docs describe shipped behavior. |
| An architecture diagram of the whole company | Scope the doc to the component. |

The one exception: if the repo keeps a `CHANGELOG.md` and follows a convention
such as Keep a Changelog, add the entry there in the repo's format. The
changelog is the only file where the change itself is the subject.

## Config table example

```markdown
## Configure

| Variable | Meaning | Default |
|---|---|---|
| `SESSION_STORE` | Backend that holds sessions. One of `redis`, `postgres`, `memory`. | `memory` |
| `SESSION_TTL` | Seconds a session stays valid. `0` keeps it until logout. | `3600` |
| `LOG_LEVEL` | Lowest level the logger writes. | `info` |
```

Give the unit and the meaning of the special values. Do not write "the TTL of
the session" for `SESSION_TTL` — that repeats the name and tells the reader
nothing.

## Check before you finish

- [ ] Every command runs.
- [ ] Every path and file name exists.
- [ ] Every relative link resolves.
- [ ] Names match the code exactly, including case.
- [ ] Defaults match the code.
- [ ] The same fact is not stated in two files with two values.
- [ ] No sentence describes the change instead of the state.
