# Components

One section per repo. Each says what goes in, what comes out, and what it depends on. None
of them says how to build it; that is the component repo's job, and its README should link
back here.

---

## store

The network graph model, the database that holds it, and the API in front of it. One repo
for now; see L-0002 for why, and for the rule that keeps a later split cheap.

**The model** covers two things that reference each other:

- **Network.** Segments, junctions and end nodes for at least: roads, electric grids, oil
  pipelines, natural gas pipelines, water pipelines. Directionality where it applies, on
  segments and through junctions. Enough to describe an interconnected network including
  flow. Not enough for a driving app: no lanes, no signals.
- **Evidence.** Sources, citations and claims. A citation supports a *claim* about a feature
  ("operated by Y from 1931 to 1948"), not the feature as a whole. The claim is the join
  between the two halves.

Every feature needs, as claims that can each be cited: ownership over time, and dates for
inception, permitting, construction, entry into operation, and exit from operation. Any of
those may be unknown, and an unknown stays unknown (I-0003).

**In:** writes from `cite`, `trace`, `backport` and `review`, through the API.
**Out:** reads for every other component; direct PostGIS access for QGIS users.
**Depends on:** Postgres with PostGIS. Running, backing up and recovering it is the
operator's concern, not this repo's.

**Open:**

- Where disagreement lives (two sources, two operators) and where identity lives (two
  sheets, maybe one pipeline). Legend's answer was observations plus link records (I-0005).
  The new model needs some answer, even a different one.
- Prior art worth reading before designing: the INSPIRE Generic Network Model (one
  node/link/direction abstraction across transport and utility networks) and
  OpenHistoricalMap's date tagging.
- The audit trail has to live in the database, not only in the API, because QGIS can write
  to PostGIS directly and would bypass an API-only audit.

---

## corpus

The archive the citations come from, and a search over it.

RAG over a large, authority-ranked corpus of relevant material, a pattern Tight Line has
built before, with exa (or similar) as a fallback when the corpus has
nothing.

Two deployables in one repo, because the index format is internal to both:

- **Ingest.** Mirror the trade press from Internet Archive (Oil & Gas Journal, Pipeline &
  Gas Journal, World Oil, American Gas Light Journal; roughly 1859-1961), clean the OCR,
  index with era-keyed vocabulary (`gas works` in 1887, `pipe line` in 1937; `pipeline`
  barely appears). Batch, slow, mostly run once.
- **Search.** A query API: place names, dates and an attribute in; ranked passages with
  stable source identifiers out.

**In:** Internet Archive items; later, other source classes (FPC/FERC filings for 1962
onward).
**Out:** passages with identifiers that resolve for a third party.
**Depends on:** nothing in this org.

**Open:**

- A fallback hit from exa is a live URL, and URLs rot. Fallback citations need a marker
  saying which lane produced them, and probably an archived snapshot taken when the
  citation is made.
- The Legend corpus spike's findings carry over directly: see Legend
  [`docs/corpus.md`](https://github.com/Tight-Line/legend/blob/main/docs/corpus.md).

---

## cite

Citation generation. The first thing anyone outside Tight Line will use: the research
partner asked for it first, and it moved from winter to fall because of that.

**In:** a pipeline segment and whatever is known about it (geometry, region, approximate
date, perhaps an operator).
**Out:** grounded citations to specific journal issues and pages, each supporting one claim
about one attribute, with the quoted passage.
**Depends on:** `corpus` search. Writes to `store` once it exists; until then, files.

The first version runs before `store` does, so its input and output are files (GeoJSON or
a spreadsheet in, a citation table out). Its first reviewers are the researchers who asked for
it, in whatever tools they already use. When `store` exists, those results go in through `backport`.

The Legend `locate` spike and its 776 attested items from 145 issues are the starting point.

**Open:**

- The shared pipeline runtime (run tracking, retry, resume without re-spending tokens, cost
  accounting, tracing) starts life here. It becomes a library when a second
  pipeline needs it, and not before.

---

## georef

Getting a scanned map onto the earth, and letting a person check and fix it.

**In:** scanned map images.
**Out:** a IIIF image and a georeference annotation with a stable, resolvable permalink.
**Depends on:** nothing in this org.

Three parts:

- A IIIF image server for the scans.
- A self-hosted Allmaps editor, which is also the georeferencing review tool. A separate
  review app is not planned unless the editor turns out not to be good enough.
- **Our own annotation server.** This is the real build. The Allmaps editor saves through a
  ShareDB websocket to Allmaps' API, and that server is not in their public monorepo
  (checked 2026-10-07), so self-hosting the editor means writing a compatible one.

Before georeferencing anything, look up whether Allmaps already has an annotation for the
image. On one spike sheet an existing CC0 annotation beat our own work by a factor of
seven.

**Open:**

- Whether the self-hosted editor is good enough for review. Settle it by trying it on a
  real sheet.
- Method and failure modes from the spike: Legend
  [`docs/georeferencing.md`](https://github.com/Tight-Line/legend/blob/main/docs/georeferencing.md).

---

## trace

Finding the lines on a georeferenced map.

**In:** a georeferenced map from `georef`, plus a statement of which symbols mean what
("dotted red is gas, ignore the rest").
**Out:** candidate segments, as GeoJSON or GeoPackage, written to `store` as unreviewed.
**Depends on:** `georef`, `store`.

This is an open research problem. A DARPA competition put the published state of the art at
F1 0.56 for lines, and the naive Opus approach in the spike was not usable. It runs
alongside everything else and blocks nothing; there is a go/no-go in the spring. Possibly
with a university computer science group.

Conservative by design: a missed spur is cheaper than a hallucinated one. The characteristic
error is a county boundary or railroad traced as a pipeline.

---

## backport

The research partner's existing traces and spreadsheets, into `store`.

**In:** the group's spreadsheets and traces, in whatever shape they are.
**Out:** structured records in `store`, with sources where the sheets have them.
**Depends on:** `store`.

A side quest, and a useful one: real data with real mess in it, exercising `store` and the
model early and often. It also carries `cite`'s file-era output into the database.

**Open:**

- Nobody on the Tight Line side has seen a sample yet. Get one early; it will push on the
  model in ways design won't.

---

## review

The authenticated web application where people check machine output and store the
verdicts.

**In:** a georeferenced map (from `georef`), candidate traces and machine citations (from
`store`).
**Out:** accepted, corrected or rejected traces and citations, written back to `store` with
the reviewer's identity in the audit trail.
**Depends on:** `store`, `georef`.

Two modes in one app, because they share authentication and the map shell:

- **Trace review.** A georeferenced map with its candidate traces over it.
- **Citation review.** Machine citations, approved or discarded.

This is where the project's credibility is decided: no machine-generated claim reaches a
release without passing through here.

**Open:**

- The September deck said trace review would happen in QGIS. The current plan is a web
  app. Decide before winter.
- Authentication for the research partner's users.

---

## release

The dataset as researchers get it.

**In:** `store`.
**Out:** versioned, citable GeoPackage and GeoJSON builds that open in QGIS with no plugins.
**Depends on:** `store`.

Can be developed independently once the model exists. Releases are built, never hand-edited.

**Open:**

- The dataset license, which constrains what can be released (see open questions).
- Open date bounds collapse on export; Legend decision 0011 worked out how.

---

## Not a component

- **An orchestrator.** There is no master pipeline. Each pipeline reads records in one state
  from `store` and writes the next; a human review is just a state change. See L-0003.
- **Deployment.** These repos are software. Each service ships something runnable;
  deployment configuration for any particular environment lives outside this org.
