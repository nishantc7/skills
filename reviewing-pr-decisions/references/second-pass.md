# Second pass: security and performance

This file is the brief for the second-pass agent. The main review lists the
choices a change made and does not judge them. This pass does the opposite: it
**hunts** for defects that leave no before/after pair to show, and it judges
each one. It runs in a fresh agent so it reaches its own conclusions, not the
main review's.

## What the agent gets

- The raw PR diff, from the API raw patch, test files included.
- The linked ticket, or the PR body when there is no ticket.
- The repo's agent instructions (`CLAUDE.md`, `AGENTS.md`), and any nested ones
  for the directories the diff touches. They hold the caps, timeouts and data
  rules this pass checks against.

It never gets the main review. A hunter handed the first reviewer's list tends
to confirm it.

## How to hunt

Run both checks below over every non-test file in the diff. Read past the diff
when a finding depends on code it calls: open the callee at the PR's head
commit and read it.

A **finding** is a concrete path: this input, reaching this line, causes this
outcome for this person. A pattern with no reachable input is not a finding.
Drop it.

A finding that something is **missing** (no limit, no signature check, no
index) needs two checks. Search for it, and open the file and read it. One
empty search is not enough: a search can miss a file, and a truncated output
can hide the line.

## Security check

Run it when the diff adds or changes a route, a login or role check, input from
a request, webhook or file, a database query, a shell or file-path call, an
outgoing URL, a secret, a log line or an analytics event. Otherwise print
`not run: the diff touches none of these`.

Hunt for:

1. **Input that reaches a sink unchecked.** Request, webhook or file data that
   reaches a database filter, a shell command, a file path, a template, an
   outgoing URL or a money amount with no validation. Name the sink.
2. **Whose data.** A user id, account id or org id taken from the request body,
   path or header where the verified login should supply it. A read or write
   that can reach another user's rows.
3. **Secrets.** A key, token or password in code, in a config file, or in a log
   line or error message.
4. **Personal data and amounts in logs and events.** Names, phone numbers,
   emails, government id numbers, account numbers or money amounts in a log
   line or an analytics event property.
5. **Unverified callers.** A new webhook or callback that does not check the
   sender's signature, or a new route on a group with no login check.
6. **Errors that say too much.** A response that returns an internal error,
   stack, query or another user's data to the caller.

## Performance check

Run it when the diff adds or changes a loop, a database query, a call to
another service or an outside API, a batch, a cache, or a scheduled job.
Otherwise print `not run: the diff touches none of these`.

Hunt for:

1. **A call inside a loop.** A network call or a database query per item. Say
   how many calls one request makes, as a formula of the input size.
2. **Calls in sequence that could run together.** Independent calls awaited one
   after another on a request path. Each adds its full latency.
3. **No size limit, or a size above the receiver's cap.** A query with no
   limit, a batch sized to the input length, or a batch larger than the other
   service accepts. Look for known caps in the repo's agent instructions and in
   the receiver's validation code.
4. **A new query with no index.** A new filter or sort on a table or collection
   where no index covers it. Name the table and the fields.
5. **Cache.** A new cache with no lifetime, a key that misses the user or a
   version, or a removed cache on a hot path.
6. **Timeouts that multiply.** A per-call timeout inside a loop or a sequence,
   so the request's worst case is the timeout times the count.

**Every performance finding names the number it depends on.** The largest
realistic input, the call rate, or the other service's response time. Write
that number as `not measured` and name where it could be measured. Never
estimate it. A guessed number reads as a fact to the human.

## Evidence

Each finding carries exactly one label, as in the main review:

- `measured in production`: name the query or the log window.
- `tested on staging`: say against what.
- `read the code`: no runtime evidence.
- `not checked`: name what nobody confirmed.

This pass reads code. It makes no call to a production service, and no write
anywhere. A finding that needs a production number to confirm says so and
stops.

## What to return

Return plain text in this shape, one block per finding, most severe first:

```
CHECK: security | performance
HEADING: one sentence that says what is true, 12 words or fewer
WHERE: path/to/file.ts:123
CODE: the 1–8 lines at WHERE, verbatim from the diff
WHAT HAPPENS: one sentence: input, path, outcome, for whom (25 words)
DEPENDS ON: performance only: the number, then "not measured" and where to measure it
EVIDENCE: one label, then up to 15 words naming the file, command or gap
```

For a check that ran and found nothing, return
`CHECK: <name> — nothing found`. For a check that did not run, return its
`not run` line.

Write the headings and sentences in plain English, by the rules in the main
skill's "Write it in plain English" section. Code, paths and identifiers stay
exact.
