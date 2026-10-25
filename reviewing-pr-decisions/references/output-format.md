# Output format

Section order in the rendered page. Every section is required. A section with
nothing in it prints its one-line empty state, not an omission.

Every heading below is the exact wording to use. They are written in plain
English on purpose; do not retitle them. `example-review.html` shows every
section filled in.

## 1. Header

PR title as the page heading. Below it: PR number, repo or package, additions
and deletions, branch, linked ticket id (or `no linked ticket found`).

## 2. What could go wrong

One sentence, from step 1: the worst realistic outcome if this change is wrong.
This is the first thing the human reads, and it frames everything below.

## 3. Scope note

Fixed text, so the human knows what the silence means:

> I skipped style, naming and test coverage. I am not saying whether to merge.
> The questions below are yours to answer.

## 4. What this touches outside its own code

One row for every point where the change reaches past its own code: an external
API, a database or schema change, a new setting or secret, a message or job, a
call to another internal service, or a changed function that code outside the
diff calls. Columns:

| Kind | What | New or changed | Where | In the approved design? | Confirmed how? |
|---|---|---|---|---|---|

- **Kind** — External API, Internal service call, Database, Config and secrets,
  Messages and jobs, or Shared code.
- **Shared code rows** — "What" names the function, its callers outside the
  diff and their count, grouped by service. "Confirmed how?" names the search
  and says each hit was read. A search alone misses a function passed as a
  value. Several changed functions with one caller each can share one row.
- **In the approved design?** — `yes` with the ticket or design id, or `no`.
- **Confirmed how?** — the same four values as a card's "How I know", or
  `docs only` when the only source is vendor documentation.
- Highlight a row (`class="flag"`) when the design did not name it, or when it
  is confirmed by `docs only` or `read the code` and can send money, data or
  messages out.

This section is never capped and never collapsed. If the change touches nothing
outside its own code, print `Nothing. The change stays inside its own code.`

## 5. Which files need you

Heading: `<N> of <total> files need you`.

**Carries a decision** — a table of file path and one line of why.

**Routine (collapsed)** — every other changed file by name, each with its
one-word reason. Collapsed by default. The human scans this to catch a wrong
skip, so a file must never be missing from either list. A migration is never
routine.

## 6. What happens, step by step

One flowchart pair first, then pseudocode pairs of the real paths. Not prose.

**The flowchart pair.** One pair per page, for the core path only.

- Two Mermaid blocks (`<pre class="mermaid">`) side by side, stacked on phones:
  `Now on <base branch>` and `With this change`. Then the fixed legend line:
  "Thick outline: a hop the change adds. Dashed outline: a hop it removes."
- Boxes are hops: a route, a function, a store, an outside API. Label each in
  plain words that say what the hop does, in quotes: `B["Wait and send it
  again"]`. The pseudocode below carries the real names.
- Show a loop as a group box, `subgraph each["For each item"] … end`, not as a
  diamond with an arrow back. Keep the group title to three or four words, or
  it is cut off.
- Mark each changed hop: `:::added` on a hop or check the change adds (right
  side), `:::removed` on one it takes away (left side). Leave the rest
  unmarked. Keep the template's two `classDef` lines in each block.
- Keep each side under about eight boxes. A path that needs more is shown in
  the pseudocode only.
- For net-new code, the left side is one box: what happened before, or
  `no prior path`.
- Do not put `<br>` inside a block. The HTML parser removes it from the text
  that Mermaid reads.

**The pseudocode pairs.** For each core path the change touches, a short name,
then two blocks side by side: `Now on <base branch>` and `With this change`.

Write each block as **pseudocode of the real path**: the trigger,
what it reads, what it calls, what it writes. Keep each block under about ten
lines. Keep the comments that matter (`// no key`, `// the default`). Wrap every
new or changed line in `<span class="n">…</span>` so it renders shaded.

Add a second path only when the change makes two paths that used to agree now
differ. For net-new code, the left block reads `new behaviour — no prior path`.

## 7. What changes for people

Where the change alters what a signal *means*, first a small table:
`Question | Before | After`, two or three rows.

Then four rows, one line each: **Better off**, **Worse off**, **Now fails
quietly**, **Now depends on**. A row with a genuine nothing prints `none`.

## 8. What you need to decide — every decision, ranked

One card per decision, ranked. With more than five cards, group them by path:
a sub-heading per path, using the same number and name as in section 6
("1. A customer pays at checkout"), then that path's cards. Cards that belong to
no path go last, under "Across the change". Number the cards in page order.

Each card:

**Kind tag** — a small label above the heading: the taxonomy category, then a
few words, for example `Numbers · rounding` or `Edge case · retry`.

**Heading** — one sentence saying what is true, in words the reader knows cold.
Not a noun phrase, not a category label, not the file name. "A charge
that timed out may have gone through, and the retry charges again", never
"Retry without idempotency".

**The code pair** — one block:

```ts
// BEFORE
await httpClient.post("/v1/charges", body);

// AFTER
await httpClient.post("/v1/charges", body, { attempts: 3 });
```

Only the changed lines. Keep enough context to read, never a whole function.
For new code, `// BEFORE — did not exist`. A comment in the pair may quote the
PR body or name a file and line.

**The question** — one question, answerable yes or no, in plain words.

> A timeout does not mean the charge failed. Does the provider reject a
> repeated charge that carries no idempotency key?

**How I know** — a coloured chip, exactly one of `measured in production`,
`tested on staging`, `read the code`, `not checked`, followed by the file,
command or gap. `not checked` says exactly what nobody confirmed.

**This is wrong if** — one line, in terms of what happens to a person.

**The other option** — optional, only where there was a real fork.

## 8b. Second pass: security and performance

The step-0 agent's findings, directly under the cards. Never collapsed. One sub-heading per check, **Security**
then **Performance**, each holding its findings as returned, or its one-line
empty state: `nothing found` or the `not run: …` line. Before the agent
returns, the whole section is one line: `still running`.

Each finding is a short card, ranked `S1`, `S2` … and `P1`, `P2` …:

**Heading** — the agent's sentence.

**Where and the code** — `file:line`, then the lines as one block. A finding
shows the code as it is after the change, not a before/after pair.

**What happens** — one sentence: input, path, outcome, for whom.

**Depends on** — performance only: the number it needs, marked `not measured`,
with where to measure it.

**How I know** — the evidence chip, as on a decision card.

Add "Also card 0N" under the heading when the finding repeats a decision card.

## 9. The ticket against the code

What the ticket asked, what the code does, and the gaps. Silence gets its own
sub-heading, **The ticket did not say**, with one line per unspecified thing the
code chose anyway.

## 10. Smaller things I noticed (collapsed)

Everything true that is not a decision. One line each, no code pairs.

---

## The length contract

Every field has a budget. A field is finished when it is inside its budget and
says the thing. These are not targets to approach; they are the shape.

| Field | Budget |
|---|---|
| What could go wrong | 2 sentences, 30 words |
| Outside-code row, "What" cell | 20 words |
| "Why you need to read it", per file | 12 words |
| Flowchart, per side | about 8 boxes |
| Each pseudocode block | 10 lines |
| What changes for people, per row | 1 sentence, 20 words |
| Card heading | 1 sentence, 12 words |
| Code pair | 12 lines total, both halves and comments included |
| The question | 1 question, 25 words |
| How I know | the chip, plus 15 words naming the file, command, or gap |
| This is wrong if | 1 sentence, 20 words |
| The other option | 1 sentence, 20 words |
| Smaller things, per line | 20 words |
| Second-pass heading | 1 sentence, 12 words |
| Second-pass code | 8 lines |
| What happens | 1 sentence, 25 words |
| Depends on | the number, plus 15 words naming where to measure it |

A card that needs more than this holds two decisions. Split it, or move the
second half to "Smaller things I noticed".

## Card writing rules

- The code pair is the payload. Never replace it with prose that describes it.
- One card, one decision. Two assumptions in one card means two cards, or one of
  them belongs in "Smaller things I noticed".
- Do not restate the PR body. Where the body is the only source for a claim,
  quote it and label the basis.
- No card may contain the words "should", "recommend", or "consider". Cards ask;
  they do not advise.

## Chip classes

| How I know | Class |
|---|---|
| `measured in production` | `chip chip-measured` |
| `tested on staging` | `chip chip-exercised` |
| `read the code` | `chip chip-reasoned` |
| `not checked` | `chip chip-unverified` |

## In CI or with no browser

Emit the same section order as GitHub-flavoured markdown, with the code pairs as
fenced blocks and the collapsed sections as `<details>`. Post it as one PR
comment. Keep every card inside its length budget. A comment that always appears
and runs long stops being read. If a change has no real decisions, post
one line saying so.
