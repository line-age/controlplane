# Roadmap

Three quarters. The shape comes from the September proposal to the research partner, with one
change: citation generation moved from winter to fall because it is what they need first.

## Fall 2026 - highest-value pieces first

- **`cite`, first.** File in, file out, before `store` exists. The research partner reviews.
- **`corpus`** ingest and search, because `cite` cannot run without it.
- **`store`**: the network + evidence model, the database, the audited API. Model work done
  jointly with the research partner. Done when a QGIS round trip against the hosted database works.
- **`georef`**: IIIF serving, self-hosted Allmaps, our annotation server. Needs time from the
  research partner on the mapping stack.
- **`backport`**: as soon as there is a sample spreadsheet, and against `store` as soon as
  `store` takes writes.
- **`trace`**: first vision models. Research; blocks nothing.

## Winter 2027 - the parts only this project needs

- **`review`**: trace review and citation review.
- **`georef`**: decide whether the Allmaps editor is good enough as the review tool.
- **`backport`**: the existing traces and metadata, all of it.
- **`release`**: the first build that opens in QGIS.
- **`trace`**: keep refining. Still not a blocker.

## Spring 2027 - hand off and wrap up

- Generalize to other network types, if fall and winter did not already.
- Move hosting to the research partner, if that is the right home.
- Go or no-go on tracing automation.
- Clean up for MIT release.

## Who

- Tight Line: research and engineering on everything, hosting during the research phase, a
  costed estimate of inference spend.
- The research partner: the first users of `cite`, joint work on the model and the mapping
  stack, the sheets and traces they already make, ground-truth citations, and their read on
  what the system proposes.
