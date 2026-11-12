# Roadmap

Three quarters. The shape comes from the September proposal to the research partner, with two
changes: citation generation moved from winter to fall because it is what they need first,
and in November `store` and `backport` moved to the front because the work turned out to be
a loop around `store`, not a line ending in it (L-0013).

## Now, to the end of 2026 - get something stored

- **`store`**, first: the model subset the natural gas traces need, the database with its
  audit trail, PostGIS views QGIS can animate, and the write API. Done when the natural gas
  traces round-trip: GeoPackage in through `backport`, GeoPackage out through `release`,
  same lines and dates, opened in QGIS against the hosted database.
- **`backport`**: the stitched natural gas traces, as the first real data in `store`.
- **`release`**: the smallest export that proves the round trip.
- **`cite`**: keep going in file mode, so its output is ready to `backport` as soon as
  `store` takes writes.
- **`corpus`** ingest and search, because `cite` cannot run without it.
- **`review`**: start on the read-only half (stored sections over their sheet, with a time
  slider and their claims), so there is something to iterate on as soon as data arrives.
- **`georef`**: IIIF serving, self-hosted Allmaps and its APIs. Needs time from the
  research partner on the mapping stack.
- **`trace`**: first vision models. Research; blocks nothing.

## Winter 2027 - run the loop

- **`review`**: the write half. Accept, correct, reject; confirm or reject proposed links;
  type in a claim and its citation; ask for citations on selected sections.
- **`cite`**: citation requests, answered at once from "Ask AI" in `review` or in bulk in
  the background, reading what is already accepted.
- **`backport`**: the research partner's spreadsheets, and the rest of their traces.
- **`release`**: regular snapshots while the loop runs, each one citable.
- **`georef`**: decide whether the Allmaps editor is good enough as the review tool.
- **`trace`**: keep refining. Still not a blocker.

## Spring 2027 - hand off and wrap up

- Generalize to other network types, if fall and winter did not already.
- Move hosting to the research partner, if that is the right home.
- Go or no-go on tracing automation.
- Clean up for MIT release.

## Who

- Tight Line: research and engineering on everything, hosting during the research phase, a
  costed estimate of inference spend.
- The research partner: the first users of `cite` and of the `review` loop, joint work on the
  model and the mapping stack, the sheets and traces they already make, ground-truth
  citations, and their read on what the system proposes.
