---
name: reviewing-pr-decisions
description: Use when reviewing a pull request whose diff is mostly routine and whose few real judgment calls are buried — "review PR #123", "review this for me", "self review", "what did the agent decide here". Lists every judgment call the code made as a before/after code pair with one question, and renders the result as an HTML page. Not a lint pass and not a test-coverage report; its only bug hunt is a security and performance second pass. Never issues a merge verdict.
---

# Reviewing PR decisions

A pull request written by a coding agent is mostly correct and mostly boring.
Forty files change, thirty-five of them mechanically. Somewhere in the other
five, the agent met an ambiguous requirement and **chose**. It chose a number, a
default, a filter, a fallback, an order of writes. Nothing in the ticket told it
the choice was wrong, so it wrote a confident PR body and moved on.

Your job is to find those choices and hand them to the human. You are not the
judge. The human has the product context you do not.

## The one rule

**Extract, don't judge.**

The agent that made the decision also validated it. That loop is why the bug
survives to the PR. You break the loop by listing every judgment call **without
ruling on it** — especially the ones that seem fine. Seeming fine is the failure
mode, not the all-clear.

For each call your output is: the code pair, the question, how it is known, and
when it would matter. Then stop. The human rules.

**Never issue a merge verdict.** Not "safe to merge", not "LGTM", not "approve",
not "no blockers" — not even when asked directly. If asked, answer with the
calls the human needs to make instead.

## The PR body is a claim, not evidence

The same agent wrote the code and wrote the PR body. A PR body that says
"verified", "breaks no caller", or "out of scope" is **the author's hypothesis
restated in confident prose.** Treat it as an input to review, never as a
finding already closed.

- A "verified" claim is only as good as the command it names. If the body names
  no command or file, it is `read the code`, whatever the body says.
- **A sentence in the PR body that limits the change is a finding, not a
  dismissal.** "This answers X, not Y" means the author knowingly left Y open.
  Surface Y as a decision with the body's own sentence quoted.
- A confident tone is not evidence. Where the body reasons from code and calls
  it proof, mark it `not checked` and say what nobody confirmed.

## Write it in plain English

The reader scans this page in two minutes, often between other things.

1. **Lead with the answer.** The first sentence of any block gives the result.
   Someone who stops after one sentence must still have the point.
2. **Use the reader's vocabulary.** Choose the common word. Define a project
   term the first time, in the same sentence. Use one word for one thing.
3. **Say each point once, as a statement of what is true.** No sentence that
   announces, restates, or ranks another sentence.
4. **Write whole sentences.** Subject, verb, object. One idea each. No noun
   stacks: "Advance and side-effects are not atomic" is three nouns doing the
   work a verb should do.
5. **Match length to the question.** To shorten, drop whole points, never
   sentence halves.
6. **A caveat earns its place only when it changes a decision.** One sentence,
   directly after the claim it limits.
7. **A heading says what is true, not what category it is.** "The payment is
   matched by order id, not by user" is a heading. "Correlation is not scoped to
   a user" is a label for people who already know the answer.
8. **Working shorthand stays behind.** Not "delta", "basis", "atomic",
   "load-bearing", "idempotent", "fan-out", "surface", "blast radius". Say what
   happens instead.

Code, file paths, status names and identifiers stay exact. The prose around
them is what gets simplified.

**Before you publish, reread the page as someone who has not seen the diff.**
Rewrite every sentence they would read twice, and every heading they could not
explain back to you.

## What this review deliberately does NOT do

Say this at the top of the output, so the human trusts the signal is high:

- No lint, formatting, naming, import, or style notes. CI catches those.
- No test-coverage commentary. **Test files are excluded from the diff entirely.**
- No generic "consider adding error handling" filler. Only concrete choices the
  code actually made.
- No merge verdict.

The one hunt is the step-0 second pass, for security and performance defects.
It reports in its own section, and the cards stay questions.

If every real item is already covered by a linter, the correct output is a
near-empty review saying so. Volume is the enemy.

## Inputs

You get a PR number or URL. Gather two things.

### The change

```bash
gh pr view <PR> --json title,body,headRefName,baseRefName,files,additions,deletions,url
gh api repos/{owner}/{repo}/pulls/<PR> -H "Accept: application/vnd.github.v3.diff"
```

`gh api` fills `{owner}` and `{repo}` from the current checkout. For a PR in
another repo, take them from the PR URL.

Use the API raw patch, not `gh pr diff`. `gh pr diff` can render a rich indented
diff whose added lines do not match a `^\+` search, which reads as "this PR adds
nothing".

If neither path authenticates, **stop and say so.** Ask for `gh auth login` or
for the diff to be pasted. Never review a reconstructed diff. An invented
finding costs more trust than no review.

### The intent

Find the linked issue. Look in the branch name, the PR title, then the body
(`Closes #123`, `Fixes PROJ-456`, an issue URL). Read it with whatever tracker
is available: `gh issue view` for GitHub issues, or a connected tracker tool
(Linear, Jira) for other ids. Take its description and acceptance criteria.

If no issue is discoverable or readable, fall back to the PR body, and print
`no linked ticket found` in the output so the missing yardstick stays visible.

## Workflow

One PR at a time.

### 0. Start the second pass

The steps below list choices and never judge them. Security defects and
performance under load often leave no choice to list: an input nobody
validated, a call inside a loop. A second pass **hunts** for those.

As soon as you have the diff and the ticket, dispatch one fresh background
subagent. Give it:

- the path to `references/second-pass.md`, as its brief;
- the raw diff saved to a file, and the PR's head commit. A squash-merged
  PR's head is reachable only after `git fetch origin pull/<PR>/head`, so
  fetch it before you dispatch;
- the ticket text, or the PR body when there is no ticket.

Give it nothing from your own review, before or after you write it. Skip the
dispatch only when every changed file is a test, a doc, or a lockfile. Then
print `not run: no code changed` in the page's second-pass section.

If your harness cannot start a subagent, run the brief yourself after step 9,
and label the section `run by the same agent as the review`.

Carry on with step 1 while it runs. Step 9 says how its result reaches the page.

### 1. Say what could go wrong, before you read the logic

Write one line: **what is the worst realistic outcome if this change is wrong?**
Derive it from the code path in front of you — an irreversible external write, a
wrong number shown to a user, data returned for the wrong user, a route that
stops answering. Do not use a fixed category list; infer it from what this diff
touches.

That line is the ranking key for step 8. Write it before you have opinions about
the code, or you will rank by what looks scary instead of by what costs most.

### 2. Triage every changed file

Split the file list in two.

**Carries a decision** — the code whose *behaviour* a human should reason about.

**Routine** — everything else. Each routine file gets a one-word reason:
`test`, `generated`, `lockfile`, `vendored`, `formatting`, `boilerplate`,
`config-value`, `rename`, `comment`.

Both lists go in the output. The routine list is collapsed, but it is **listed
by name**, so the human can catch a wrong skip.

A migration is never routine.

Then list **what the change touches outside its own code**: every external API,
database or schema change, new setting or secret, message or job, and call to
another internal service. A changed function that code outside the diff calls
also gets a row, of kind **Shared code**. Name its callers and their count. Find
them by searching the whole repo (every service, in a monorepo), and read each
hit. For each row, record whether the ticket or design named
it, and how it was confirmed (`docs only` counts, and is weak). This list is
never capped and never collapsed, because a call to an outside system is where
an unverified assumption costs the most.

### 3. Trace the flow, before and after

Trace the core path twice: at the merge base, and on the head branch. What
triggers it, what it reads, what it computes, what it writes, what it returns.

Open the section with one flowchart pair: the core path as boxes, before beside
after, so the reader sees the shape of the change at a glance. Mark each hop
the change adds, and each hop it removes. Then write each side as short
pseudocode of the real path, and mark the new lines. The flowchart never
replaces the pseudocode. The rules for both are in `references/output-format.md`
section 6.

Tracing one side cannot tell you whether the change helped. The pair shows the
human what they are trading. It also exposes the "fetch from A, hand to B, then
re-read from B" class of waste, and safety checks the old path had that the new
one lost.

When the change makes two paths that used to agree now differ (a new route that
skips a shared service, for example), trace both.

For net-new code with no prior path, write `new behaviour — no prior path`
rather than inventing a before state.

### 4. Say what changes for people

One line each, in user terms, not code terms:

- **Better off** — which user, in which situation, gets a better outcome.
- **Worse off** — which user, previously served, is now served worse. A narrower
  timeout, a stricter filter, a new gate, or a new cap almost always has one.
- **Now fails quietly** — behaviour that used to fail loudly and now degrades
  quietly. Log-only is silent from the user's side.
- **Now depends on** — code that does not appear in the diff but that the change
  now leans on harder.

Where the change alters **what a signal means** rather than what a step does,
lead with a small before/after table of the questions at stake — *what marks
this paid? what stops a second write?* Two or three rows. If one row's Before
column repeats another's, one signal was doing two jobs.

### 5. Extract the decisions

Walk the logic against `references/decision-taxonomy.md`. List **every judgment
call**, seeming-fine ones included. Tag each with its taxonomy category, so the
human sees the kind of problem before reading the card. Each renders as four
parts, and each part has
a word budget — **the length contract in `references/output-format.md` is part
of the format, not advice.**

**a) The code pair.** One block, `// BEFORE` above `// AFTER`, only the lines
that changed. **The pair is the payload. Do not paraphrase it into prose.** A
one-word change can be the entire bug, and the human can only see that from the
two lines together. For new code write `// BEFORE — did not exist`.

**b) The question.** One direct question naming the assumption the code rests
on, answerable yes or no. If a comment or a variable name asserts the
assumption, quote it.

**c) How I know.** Exactly one of:
- `measured in production` — name the query or the log window
- `tested on staging` — say against what
- `read the code` — no runtime evidence
- `not checked` — name precisely what nobody confirmed, and what breaks if the
  assumption is wrong

**Never leave this blank and never upgrade a guess.** A `not checked`
assumption on a boring line outranks a well-tested dangerous one.

**d) This is wrong if.** The condition under which the choice is wrong, in terms
of what happens to a person.

`The other option` is optional. Include it only where there was a real fork.

### 6. Compare the ticket against the code

Flag any gap, including "does X correctly, but by a path the ticket did not ask
for".

**Where the ticket was silent and the code chose anyway, the silence is the
finding.** Say what the ticket did not specify, then what the code picked.

### 7. Check repo discipline

Read the repo's agent instructions (`CLAUDE.md`, `AGENTS.md`, or a
`CONTRIBUTING.md`) and any nested ones for the changed directories. Defer to the
project; do not hardcode rules here. Flag layer skips, logic that leaked into
the wrong layer, and any path that could read or write across users or tenants.
Cross-user leakage always ranks first.

### 8. Rank every decision

List every decision as a card. A large PR has many, and each one is a call the
human must make. Order them by:

1. Anything marked `not checked` that the step-1 answer depends on.
2. **New** code that can reach the step-1 outcome where the old path could not.
3. Numeric, unit, and threshold decisions that gate behaviour.
4. Decisions the ticket was **silent** on.
5. Behaviour a user loses, or that became silent.
6. Layer or tenancy violations, then efficiency with real latency or cost.

Two cards that ask the same question become one card. A true point that is not
a decision, such as a stale comment, goes in a collapsed "Smaller things I
noticed".

### 9. Emit the review

Publish as soon as steps 1–8 are done; the cards never wait for the second
pass. If the step-0 agent has already returned, its findings go in this
publish. If it is still running, the "Second pass" section reads `still
running` and you publish anyway. When it returns, fill the section and publish
again to the same place. That is the one republish.

Put its findings in the order it returned them, as returned. Where one repeats
one of your cards, add "Also card 0N" and keep both.

Fill `assets/review-template.html` and write it to
`<temp dir>/pr-<number>-review.html`.

- If your harness can publish an HTML page (for example an Artifact tool),
  publish the file and give the human the link and the local path.
- Otherwise add the two lines that the template's top comment gives (the
  doctype, and the charset and viewport tags), then open the file in a browser (`open` on macOS, `xdg-open` on Linux). Give the human the path.
- In CI, or with no browser, follow "In CI or with no browser" in
  `references/output-format.md`.

**Never dump the review inline in the terminal.** The side-by-side code pairs
are the point, and they do not survive as chat text.

## Red flags — stop and re-read this skill

| You are about to write | What it means |
|---|---|
| "Looks good", "LGTM", "safe to merge" | You judged. Delete it and list the calls instead. |
| "Good test coverage" | Tests are out of scope. Cut it. |
| "The PR body says this was verified" | That is a claim. Find the command, or mark it `not checked`. |
| "The author already noted this is out of scope" | That is a finding. Surface it with the sentence quoted. |
| "This one is obviously correct, no need to surface it" | That thought is the signal. Surface it. |
| Two cards that ask the same question | Merge them into one card. |
| "The change moves a route / renames a group, so there is no logic" | Where a request is routed **is** logic. Trace both sides. |
| A finding with no `// BEFORE` / `// AFTER` block | You paraphrased. The pair is the payload. |
| A card longer than one screen | It holds two decisions. Split it, or cut to the budget. |
| A heading that is a noun phrase, not a sentence | Rewrite it as a statement a reader understands cold. |
| A flowchart or prose in place of the pseudocode pairs | The flowchart pair sits above the pairs; it never replaces them. Write the real path as pseudocode. Shade the new lines. |
| A changed function with callers outside the diff, and no "Shared code" row | Search the whole repo for its callers, read each hit, and give it a row with the count. |
| A new external API call mentioned only inside a card or the flow | It belongs in the "outside its own code" table, with how it was confirmed. |
| "The docs say the provider supports this" | Docs are a claim. Mark it `docs only` until someone calls it. |
| A brief to the second-pass agent that mentions your cards | You anchored it. Send only the diff, the ticket and `references/second-pass.md`. |
| A second-pass finding you reworded, merged or dropped | Its findings go on the page as returned. Add "Also card 0N" where they repeat a card. |

## References

- `references/decision-taxonomy.md` — the decision categories to walk in step 5.
- `references/output-format.md` — section order, card format and length budgets.
- `references/second-pass.md` — the security and performance brief for the
  step-0 agent.
- `assets/review-template.html` — the page to fill in step 9.
- `example-review.html` — a filled-in review of an invented PR. It is the
  quality bar. Read it before writing your first review. Its blue notes explain
  each section; a real review has no notes and no banner.
