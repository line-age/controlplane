# Decisions

Two lists. **L-** decisions were made for Lineage. **I-** decisions are inherited from the
Legend spike: they are advisory, written down so the reasoning is not lost, and any of them
can be reopened. Legend's full log is
[`docs/decisions.md`](https://github.com/Tight-Line/legend/blob/main/docs/decisions.md).

Each entry is short on purpose. Add the reasoning when a decision is reopened, not before.

---

## Made for Lineage

### L-0001 - The data lives in a database, not in git

*2026-10-07.* Git is unwieldy for a large corpus of individual segments and citations, and
an API lets web tools and AI integrations work with the data without knowing the mechanics
of git.

The cost: someone has to host the database reliably and back it up.

What git was doing has to be rebuilt deliberately: an append-only history of who changed
what and why. Releases stay versioned snapshots. Replaces Legend 0003, 0007, 0012 and the
git half of 0016.

### L-0002 - Model, database and API start in one repo

*2026-10-07.* The network model and the evidence model are one schema with two namespaces,
behind one API, in `store`. Citations support claims about features, so the two halves
are joined at the claim. Two services with cross-references would mean cross-service
referential integrity.

The rule that keeps a later split cheap: the model sits in its own top-level directory and is
versioned separately, and other repos depend on the published model package or the API,
never on the migrations.

### L-0003 - Several pipelines, no orchestrator

*2026-10-07.* `cite`, `georef` and `trace` are separate pipelines. They chain through record
states in `store`, not through a master DAG. A shared runtime library gets extracted when a
second pipeline needs one.

### L-0004 - Georeferencing review is the Allmaps editor

*2026-10-07.* Unless trying it shows otherwise. The build is our own annotation server,
because Allmaps' server is not public.

### L-0005 - Citations first

*2026-10-07.* `cite` moved from winter to fall at the research partner's request. It ships before `store`,
with file input and output.

### L-0006 - Software here, deployment elsewhere

*2026-10-07.* These repos hold software. Each service ships something runnable, such as a
container image; configuration for any particular deployment lives outside this org. Plain
PostGIS behind the API and nothing tied to one environment, so whoever runs Lineage can
run it anywhere.

### L-0007 - Names

*2026-10-07.* The project is **Lineage**. The GitHub org is **`line-age`**.

---

## Inherited from Legend (advisory)

### I-0001 - Absent attributes stay absent

Never synthesize a value to fill a field. Realistic input yields geometry, an approximate
year, sometimes an operator, and nothing else. (Legend 0002.)

### I-0002 - No citation, no machine-generated claim

Citation identifiers must resolve when the claim is made. Fabricated sources are the failure
that would end the project's credibility. (Legend 0004.)

### I-0003 - The map-date rule

A line on a 1930 map proves existence *by* 1930, not construction *in* 1930. This should be
code, not a prompt. (Legend Phase 4 design.)

### I-0004 - The system never decides identity

When a new map shows a line that may already be in the data, the system proposes the link
with evidence and a person confirms it. A silently fused pair of distinct pipelines is worse
than a duplicate, because nobody ever notices it. (Legend 0005.)

### I-0005 - Facts and judgements kept apart

What one source said is a fact and does not change. Which pipeline it belongs to is a
judgement and may. Legend kept them in separate records. (Legend 0012, 0016.)

### I-0006 - License lanes recorded at ingest

ODbL (OSM), CC BY 4.0 (GEM) and US public domain do not compose, and provenance cannot be
recovered later. (Legend 0006.)

### I-0007 - No nodes unless a source shows them

Most historical sheets show lines only. Inventing junctions records inference as
observation. Worth revisiting now that the model has junctions in it. (Legend plan.)

### I-0008 - MIT for the code

The dataset license is a separate question and still open. (Legend 0009.)

### I-0009 - Pre-1920 is works and transmission, not town mains

Town distribution routes are not in any source; recording them means inventing geometry.
Probably needs revisiting for water. (Legend 0017.)
