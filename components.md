# Components

One section per repo. Each says what goes in, what comes out, and what it depends on. None
of them says how to build it; that is the component repo's job, and its README should link
back here.

---

## store

The network graph model, the database that holds it, and the API in front of it. One repo
for now; see L-0002 for why, and for the rule that keeps a later split cheap.

**The model** covers two things that reference each other:

- **Network.** Edges, junctions and end nodes for at least: roads, electric grids, oil
  pipelines, natural gas pipelines, water pipelines. Directionality where it applies, on
  edges and through junctions. Enough to describe an interconnected network including
  flow. Not enough for a driving app: no lanes, no signals.
- **Evidence.** Sources, citations and claims. A citation supports a *claim* about a feature
  ("operated by Y from 1931 to 1948"), not the feature as a whole. The claim is the join
  between the two halves.

Every feature needs, as claims that can each be cited: ownership over time, and dates for
inception, permitting, construction, entry into operation, and exit from operation. Any of
those may be unknown, and an unknown stays unknown (I-0003).

`store` is the middle of the loop (L-0013). Every other component either fills it, works
on what is in it, or builds from it, and all of them can run at once against the same
data. That makes it the first thing to build, with `backport` as its first writer.

**In:** writes from `cite`, `trace`, `backport` and `review`, through the API. Citation
requests, from a person in `review` or from a bulk job.
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
- Which unit claims attach to. A diameter or a build date is a fact about a section; flow
  and connectivity are facts about edges. The two don't line up: a branch splits an edge but
  not a section, and a later extension adds a section but maybe not an edge.

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

**In:** a pipeline section and whatever is known about it (geometry, region, approximate
date, perhaps an operator).
**Out:** grounded citations to specific journal issues and pages, each supporting one claim
about one attribute, with the quoted passage.
**Depends on:** `corpus` search. Writes to `store` once it exists; until then, files.

The first version runs before `store` does, so its input and output are files (GeoJSON or
a spreadsheet in, a citation table out). Its first reviewers are the researchers who asked for
it, in whatever tools they already use. When `store` exists, those results go in through `backport`.

Once `store` takes writes, `cite` works from citation requests recorded in `store`
(L-0013). A request comes one of two ways: a person clicks "Ask AI" on a section in
`review`, and `cite` processes that request at once while they wait; or a bulk job files
requests for many sections ("every gas section with no owner") and `cite` works through
them in the background. Either way it reads every claim already accepted for the section,
and writes candidate claims and citations to `store` as it finds them, so a section looks
the same afterwards whichever way it was asked about. The more is known about a
section, the better the queries get, so the same section can be asked about again and
again as the loop goes on.

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

Four parts:

- A IIIF image server for the scans.
- A self-hosted Allmaps editor, which is also the georeferencing review tool. A separate
  review app is not planned unless the editor turns out not to be good enough.
- **The Allmaps APIs, self-hosted.** The editor saves to Allmaps' Live (a ShareDB
  websocket), REST and Annotations APIs. All three are in Allmaps' public monorepo, so we
  run theirs on Node against our Postgres. The real build is closing the gaps: editing
  gated to GitHub teams (L-0012), every saved version recording who saved it, and Node
  builds. Upstream first; see
  [allmaps#672](https://github.com/allmaps/allmaps/discussions/672).
- **Allmaps Tile Server, Preview and Viewer, self-hosted.** The Tile Server turns
  annotations into warped XYZ tiles, which is how a georeferenced sheet opens in QGIS.
  allmaps.org runs all three on Cloudflare; ours run in the same deployment as the rest.

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
**Out:** candidate traces, as GeoJSON or GeoPackage, written to `store` as unreviewed.
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

No longer a side quest (L-0013). `backport` is the first writer to `store` and supplies the
data the `review` loop starts from. Real data with real mess in it, exercising `store` and
the model early and often. It also carries `cite`'s file-era output into the database.

The first input is the research partner's natural gas traces (sixteen sheet-to-sheet
periods from 1929 to 1997, about 2,300 traced lines and 180 removals), already stitched into
one GeoPackage with sheet dates and proposed removal matches. It carries no attributes
beyond geometry and the sheet it came from, which is exactly the starting state the loop
expects.

**Open:**

- The research partner's spreadsheets are still unseen. The traces were the first sample;
  get a spreadsheet early, because it will push on the model in ways traces don't.

---

## review

The authenticated web application where people work on what is in `store`, continuously.
It is where the loop runs (L-0013): not a gate at the end, but the place a researcher goes
to look at sections, find out more about them, and record what they found.

**In:** sections, claims, citations and proposed links (from `store`); georeferenced sheets
(from `georef`).
**Out:** accepted, corrected or rejected claims and links, new claims and citations a
person typed in, and citation requests, all written to `store` with the person's identity
in the audit trail.
**Depends on:** `store`, `georef`; `cite` through the requests it leaves in `store`.

What a person does in it, in one map shell with one login:

- **Look.** Stored sections over the sheet they came from, with a time slider, filtered by
  what is known and what is missing ("no build date", "proposed removal").
- **Review.** Accept, correct or reject a candidate trace, claim or citation.
- **Decide identity.** Confirm or reject proposed links ("this removal ends that section",
  "these two traces are one pipeline"). The system proposes; a person decides (I-0004).
- **Ask AI.** On any section, ask `cite` for what we don't know yet, given everything we
  do. It answers while the person waits, with candidate claims and citations, never
  accepted ones. The same candidates also arrive from background bulk runs.
- **Add what they know.** A person can type a claim and its citation (a journal page, a
  filing, a sheet) directly. These are human claims and need no machine review.

Every accepted claim is input to the next request, which is what makes it a loop. This is
also still where the project's credibility is decided: no machine-generated claim is
accepted without a person accepting it here.

**Open:**

- How much of the AI help runs inside `review` and how much is `cite` on demand. Probably
  `cite` does the searching and `review` does the conversation.
- What "nothing left to add" looks like for a section, so people can see where the loop is
  done and where it isn't.
- The September deck said trace review would happen in QGIS. The current plan is a web
  app; QGIS stays available for reading, and for editing straight against PostGIS.
- Authentication for the research partner's users. GitHub login gated by team, as in
  `georef` (L-0012), is the obvious first answer.

---

## release

The dataset as researchers get it.

**In:** `store`.
**Out:** versioned, citable GeoPackage and GeoJSON builds that open in QGIS with no plugins.
**Depends on:** `store`.

Can be developed independently once the model exists. Releases are built, never hand-edited.

A release can be built from any state of `store` (L-0013): it is a snapshot of where the
loop is, with whatever citations exist at that moment. A first release can come as soon
as `backport` has loaded the natural gas traces, and should reproduce the stitched
GeoPackage that input came from.

**Open:**

- Whether a release carries candidate claims at all, and if so how they are marked. Leaving
  them out is the safe default; a research build that includes them, flagged, may be what
  the research partner wants.
- The dataset license, which constrains what can be released (see open questions).
- Open date bounds collapse on export; Legend decision 0011 worked out how.

---

## deploy

What it takes to run Lineage somewhere, without saying where.

**In:** the container images the other repos ship.
**Out:** generic Helm charts, and synthetic fixture data for development and tests.
**Depends on:** every service repo.

Generic means no hostnames, cluster names, secrets or environment values; whoever runs
Lineage supplies those from outside this org (L-0006). Fixture data is made up or drawn
from public sources, never the research partner's sheets or scans. See L-0009.

**Open:**

- Whether fixtures for a single component live here or in that component's repo.

---

## Not a component

- **An orchestrator.** There is no master data processing pipeline. Each pipeline reads records in one state
  from `store` and writes the next; a human review is just a state change. See L-0003.
- **Deployment configuration.** These repos are software. Each service ships something
  runnable, and `deploy` packages it generically; configuration for any particular
  environment lives outside this org.
