---
name: html-docs
description: Style and structure rules for standalone HTML documents such as walkthroughs, comparisons, proposals, designs, analyses, and research write-ups. Every document gets the same look (the "Paper" style), a left contents panel, a dated header with sources, diagrams with labelled arrows, status marks, and stable IDs for later reference. Use when the user asks for an HTML doc, report, write-up, comparison, proposal, design doc, walkthrough, analysis, or research summary as a page or file.
---

# HTML docs

Every document is one self-contained `.html` file built from
[`assets/template.html`](assets/template.html). The template holds the style.
This file holds the rules for what goes in it.

## Writing

Keep responses concise. Answer the question directly first, then add only the
detail needed to act on it. Avoid long preambles, repetition, and exhaustive
lists of options. The text shouldn't be overwhelming.

- Use short paragraphs of 1–4 sentences. Use one idea per paragraph.
- Use a table when you compare two or more things on the same criteria.
- Use a diagram when the reader must see flow, structure, or order. Do not add
  a diagram only for decoration.
- Do not repeat the summary in the conclusion.

## Build

1. Copy `assets/template.html` to the output path. The default path is
   `<kebab-case-title>.html` in the current directory, unless the user gives
   another path.
2. Do not change the `<style>` or `<script>` blocks. Do not add other CSS,
   JS, or external files. If a component is missing, use the nearest one that
   exists.
3. Replace every `{{placeholder}}`. Delete the example components you do not
   use, and delete the template's HTML comments.
4. Before you finish, check that no `{{` is left, that every contents link
   points to a section or `h3` `id`, and that every `xref` and `cite` points to
   an `id` that exists.

## Structure

**Header.** It is always in this order:

| Part | Rule |
|---|---|
| Type | One word: Proposal, Analysis, Comparison, Walkthrough, Research, Report, or another word that fits. |
| Title | A short noun phrase. No trailing period. |
| Subtitle | One sentence that says what the document answers or proposes. |
| Meta | `Date` is required, in ISO form in `datetime`. Add `Author` and `Status` when known. You can add other short fields, such as `Version`. |
| Sources | A numbered list. Each item has `id="src-N"`. If there are no sources, write "None: based on the author's own analysis." |

**Contents panel.** It has one link per `<section>`, in the same order, with
the same text as the `h2`, but without the number. When a section has 2 or
more `h3`, put a nested `<ol>` under its link, with one link per `h3`. When a
section has only one `h3`, do not list it.

**Sections.** Number the sections in order. The first section is always
`Summary`, and it gives the direct answer. Use `h3` for sub-parts inside a
section. Each `h3` has an `id` in the form `<section-id>-<slug>`, for example
`findings-cost`. Do not number the `h3`. Do not go deeper than `h3`.

Suggested sections for each type. Use them as a starting point, not as a
fixed list:

| Type | Sections after Summary |
|---|---|
| Proposal / design | Context · Current state · Options · Proposed design · Risks · Open questions · Decisions |
| Comparison | Criteria · Comparison · Recommendation |
| Analysis | Scope · Findings · Causes · Recommendations |
| Walkthrough | Prerequisites · Steps · Result · Troubleshooting |
| Research | Background · Key concepts · Findings · Discussion · Conclusion |

## Sources and citations

- Cite in text with `<a class="cite" href="#src-N">[N]</a>`, placed directly
  after the claim.
- Number the sources in the order they are first cited.
- In a Research document, every factual claim that is not common knowledge
  needs a citation.
- Source format: `Author, Title (linked), Publisher or site, Year. Accessed
  YYYY-MM-DD.` Leave out the parts you do not know. Never invent a source, an
  author, a year, or a URL.

## Status marks

Use only these three. The markup is in the template. Never use emoji.

| Mark | Class | Meaning |
|---|---|---|
| ✓ | `st ok` | Correct, meets the need, confirmed |
| ✕ | `st bad` | Wrong, fails the need, rejected |
| ! | `st warn` | Warning, trade-off, partly true, needs attention |

- Put a status mark in front of the text it rates, and follow it with a short
  word or phrase.
- A mark must add information. If every item would get the same mark, use no
  marks.
- In a comparison table, rate each cell. Do not rate the criterion names.

## IDs

Give an ID to an item that someone will refer to later, such as in the
document, in a review, or in a later conversation. Do not give IDs to all
paragraphs.

**Format.** `PREFIX-NN`: an uppercase prefix, a hyphen, and two digits
(`F-01`). For options, use a letter (`O-A`, `O-B`). The anchor `id` is the
same as the visible text.

**Stability.** Number IDs in the order they appear, per prefix. After the
document is shared, never renumber or reuse an ID. Add new items at the next
free number, even if the order is then not sequential.

**Cross-references.** Write `<a class="xref" href="#F-01">F-01</a>`. Refer to
an item by its ID, not by "the point above".

## Diagrams

- Use inline SVG with the template's classes. Do not use images or diagram
  libraries.
- Use rectangles for nodes. Use 1–3 words per node. Use `.sub` for a second,
  smaller line.
- Every arrow has a label. Use 1–3 words, placed at the midpoint above the
  line.
- When order matters, start each label with its step number (`1 request`,
  `2 verify`).
- Line types: a solid `.edge` is the main flow. A `.edge.dashed` is a return,
  async, or optional flow.
- Node types: `.bad` is a failure point, `.accent` is the focus of the
  diagram, `.group` is a container. You can mark a node with an ID, using
  `.node-tag` above it.
- Use left-to-right flow. Use no more than about 8 nodes. If you need more,
  split the diagram into two.
- Each `<marker>` has an `id` that is unique in the page (`arr1`, `arr2`, …).
- Each diagram has `aria-label` text and a caption `Fig. N`, with one sentence
  that says what the diagram shows.

## Callouts

Use `.callout.note` for the key takeaway and `.callout.warn` for a caution.
Use no more than one callout per section. Start with a bold word, such as
**Recommendation.**, **Note.**, or **Watch.**
