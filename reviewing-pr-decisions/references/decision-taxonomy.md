# Decision taxonomy

Walk this list against the logic in step 5. It is a checklist for *finding*
choices, not for ranking them. Ranking happens in step 8, against the stake you
named in step 1.

A category with nothing in it is a category you skip silently. Do not write
"no numeric decisions in this change" — that is filler.

Each category name is also the card's kind tag (`Numbers`, `Edge case`,
`Security`, and so on). See `output-format.md`.

## 1. Numbers and precision

- A literal that gates behaviour: a timeout, a limit, a page size, a retry
  count, a cap, a TTL, a threshold.
- Where did the number come from? If nobody measured it, the basis is
  `read the code` at best, and this is a real fork worth `The other option`.
- Rounding, truncation, and the direction of each. Who absorbs the remainder.
- Units. Seconds against milliseconds, dollars against cents, percent against
  fraction. A unit error survives every test that uses the same wrong unit on
  both sides.
- An integer where the domain is fractional, or the reverse.

## 2. Defaults and fallbacks

- A value chosen when the input is absent, empty, or null. Is the default a real
  answer, or a placeholder a user will read as a real answer?
- A fallback that silently substitutes. Does the caller learn the substitution
  happened?
- Null against zero against an empty list. These often decode identically from
  an absent JSON key, and the code usually treats one of them as meaningful.
- A hardcoded default that a config file should own, or a new setting that is
  unset in some environment.

## 3. Filtering and selection

- A new condition that drops rows. Which previously-served rows now vanish?
- Sort order, tie-breaks, and what happens when the list is empty.
- Deduplication: which duplicate wins.
- A limit on a query, and whether the caller can tell the result was truncated.

## 4. Edge cases the code decided quietly

- Empty input, single element, all elements identical.
- A first-time user with no history, against an established one.
- A concurrent second call, a retry, or a double click. What stops a duplicate
  write?
- Partial failure across several calls. Does one failed call fail the whole
  response, or return a partial one with no marker?
- Time: month and year boundaries, holidays, time zones. Business logic in local
  time on servers in UTC makes any date arithmetic a decision.

## 5. Data flow

- Where each value comes from and where it goes. Draw the hops.
- A redundant hop: fetched from A, written to B, then re-read from B in the same
  request.
- A value that crosses a service boundary. Does the receiving field name match
  the sending key? A renamed key often decodes to zero without an error, and
  tests that build objects directly never touch the name.
- Sequential calls that have no dependency and could run together. Latency is
  user-visible.
- A cache added or removed: the TTL, the key, and what a stale read looks like
  to the user.

## 6. Contract assumptions

- An assumption about a response another service returns: a field always
  present, a list never empty, an enum with a known set of values.
- A test that asserts an error string owned by another module. The string is not
  a contract; it changes under you.
- A dependency version bump, and which behaviour the new version changes.
- An environment variable read in code but absent from a deployed config, so it
  is empty in every deployed environment.

## 7. Routing, auth, and tenancy

- Which group or middleware a route sits behind, and which callers that gate
  admits. A route move **is** a logic change, even when the handler is untouched.
- Any path where the id used to read or write comes from the caller rather than
  from the verified identity.
- A gate loosened for one caller that is thereby loosened for all of them.
- Anything that could return one user's or tenant's data to another. Rank first,
  always.

## 8. Structure

- Logic that landed in the wrong layer, against the layering in the repo's agent
  instructions or contributing guide.
- A layer skipped: a controller reaching a repository, a handler reaching an
  external API.
- A rule that now lives in two places, so the next change lands in only one.
- A new abstraction with exactly one caller.

## 9. Reversibility

For each write the change can perform, ask: can it be undone from the app, from
a console, or not at all? An irreversible write reached by new code — money
sent, an email delivered, a record deleted — is the highest-value thing this
taxonomy finds. It belongs at the top of the ranking whatever the diff looks
like.
