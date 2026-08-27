# Writing in Simplified Technical English

ASD-STE100 is a controlled English standard for technical documentation. It has
two parts: a set of writing rules, and a dictionary of approved words. The
dictionary is a controlled document that this skill does not reproduce. Apply
the rules below and the substitution table. When you are unsure whether a word
is approved, choose the shorter and more common word.

## The rules that apply to software docs

### Words

- **One word, one meaning.** Use each word in one sense only. `screen` is a
  display, not a verb for filtering.
- **One meaning, one word.** Do not vary your vocabulary for style. If you call
  it a *token*, call it a *token* every time. Never *token*, then *credential*,
  then *secret*, for the same thing.
- **Keep the technical names.** Class names, flags, protocols, and file names
  stay exactly as they are. Do not simplify `OAuth2AccessToken` into "the login
  thing".
- **No slang, no idiom, no metaphor.** Not "under the hood", "out of the box",
  "a breeze", "kick off", "spin up".
- **No marketing words.** Not "powerful", "seamless", "robust", "blazing fast",
  "simply", "just", "easy". State the fact instead.

### Verbs

- **Use the active voice.** "The parser reads the file." Not "The file is read
  by the parser."
- **Use simple tenses.** Simple present for behavior, simple past for a
  completed event. Avoid stacked helping verbs: "will have been created".
- **Avoid the `-ing` form.** Rewrite "Before installing the package, check the
  version" as "Before you install the package, check the version." Technical
  names keep their `-ing` (`string`, `logging`, `caching`).
- **Start an instruction with the verb.** "Run the migration." "Set the flag."

### Sentences

- **20 words maximum** in an instruction. **25 words maximum** in a description.
- **One idea per sentence.** Split a sentence that has two ideas.
- **One instruction per sentence.** Two only when the reader does them at the
  same time.
- **Keep the articles and the connecting words.** "Set the value in the config
  file", not "Set value in config file". Keep `that` and `which`.
- **Do not stack nouns.** Three words maximum in a noun cluster. Rewrite
  "user account balance update handler" as "the handler that updates the account
  balance".
- **Six sentences maximum per paragraph.** One topic per paragraph.

### Structure

- **Use a list** when the text has more than two conditions or steps. A list is
  clearer than a long sentence with `and` and `or`.
- **Put the condition first.** "If the file does not exist, the tool creates it."
  Not "The tool creates the file if it does not exist."
- **Put the warning before the step it applies to**, never after.
- **Give the instruction, then the reason.** "Stop the service before you edit
  the database. The service holds a write lock."
- **Use digits for numbers.** "3 retries", not "three retries".
- **`must`** for a requirement. **`can`** for a possibility. **`do not`** for a
  prohibition. Never `shall`. Avoid `should` — say what happens instead.

## Substitutions

Replace the left column with the right column.

| Do not write | Write |
|---|---|
| utilize, leverage, employ | use |
| in order to | to |
| prior to | before |
| subsequent to, following | after |
| commence, initiate | start |
| terminate, cease | stop |
| attempt | try |
| obtain, acquire | get |
| provide | give |
| require | need |
| perform, execute, conduct | do, run |
| modify, alter | change |
| indicate | show |
| additional | more |
| approximately | about |
| sufficient | enough |
| numerous, multiple | many |
| assist | help |
| facilitate | help, make easy |
| in the event that | if |
| in the case of | for |
| due to the fact that | because |
| at this point in time | now |
| a number of | some, many |
| is able to | can |
| has the ability to | can |
| it is necessary to | you must |
| please note that | (delete it) |
| basically, essentially, actually | (delete it) |
| simply, just, easily | (delete it) |

## Rewrite example

Before:

> This module basically provides a powerful and flexible abstraction layer for
> facilitating the retrieval of user session data from multiple heterogeneous
> backend storage providers, allowing developers to easily swap implementations
> without needing to modify any of their existing calling code.

47 words, one sentence, passive nouns, marketing words, an `-ing` chain, and a
five-word noun cluster.

After:

> This module reads user session data. It supports more than one storage
> backend. To change the backend, you set the `SESSION_STORE` variable. The
> caller code does not change.

Four sentences, 11 words at most, active voice, and it names the actual
variable.

## Docstring style under these rules

The summary line is one sentence in the simple present tense. It says what the
function does, not how it was built.

```python
"""Read the session for a user ID."""            # good
"""Reads and returns the user's session data."""  # redundant verb pair
"""This is a helper for getting sessions."""      # says nothing
```

Parameter and return descriptions are noun phrases or short sentences. Give the
unit, the range, and the meaning of special values.

```python
Args:
    timeout: Seconds to wait for the server. 0 disables the timeout.
    strict: Raise an error on an unknown field. The default keeps the field.
```

## Checklist

Run this over every paragraph and docstring you write:

- [ ] No sentence is longer than 25 words.
- [ ] Every instruction starts with a verb.
- [ ] No passive voice.
- [ ] No `-ing` verbs outside technical names.
- [ ] One term per concept, used consistently.
- [ ] Articles are present.
- [ ] No noun cluster longer than 3 words.
- [ ] No marketing words, no idioms, no hedging.
- [ ] Conditions and warnings come before the action.
- [ ] No sentence describes an edit or a past version of the code.
