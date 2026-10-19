# Glossary

The words every repo should use the same way. Add to it when a component invents a term.

**Annotation (georeference annotation).** A IIIF-standard record of the control points that
place a scanned map on the earth. Allmaps reads and writes these.

**Attestation.** A passage in a source that says something about a feature. A citation
points at one.

**Citation.** A pointer to a specific source (journal, issue, page) that resolves for a third
party, supporting one claim.

**Claim.** One statement about one attribute of one feature: "operated by Y from 1931 to
1948". The thing a citation supports.

**Corpus.** The archive citations come from: today, the US trade press on Internet Archive,
1859-1961.

**Control point (GCP).** A point identified both on the scan and on the earth. Georeferencing
is choosing these well.

**Edge.** The stretch of a network between two nodes (junctions or end nodes): the graph
and topology term. INSPIRE's network model calls it a link. Mostly internal to `store`.

**Era-keyed vocabulary.** Search terms that depend on the period: `gas works` in 1887,
`pipe line` in 1937.

**Junction.** A node where edges meet. Recorded only where a source shows one (I-0007).

**Lane.** Which license, or which retrieval path, a piece of data came through. Recorded at
ingest.

**Lead (team lead).** The person who owns a repo and its project. Listed by GitHub handle
in the controlplane README.

**Line.** Plain English for a pipeline, transmission line, road or main as a whole. Not a
technical unit; use edge, section or segment when the unit matters.

**Map-date rule.** A line on a map published in year N proves existence by N, not
construction in N (I-0003).

**Project.** A GitHub project board in the `line-age` org; one per repo (L-0008). Not to be
confused with Lineage itself, which is "the project" in prose.

**Section.** A continuous stretch of line built and put in use at one time. The unit the
research partner's line attributes describe: one row per section per year. A section split
later becomes 1a, 1b... retroactively. Not the federal "line section" (49 CFR 192.3),
which is bounded by valves and compressor stations, not by construction date.

**Segment.** The straight piece of a line between two consecutive vertices, as in the OGC
Simple Features standard that QGIS and PostGIS follow. Geometry only; nothing is recorded
about a segment. Until 2026-10-19 this glossary used "segment" for what is now an edge
(L-0011).

**Sheet.** One scanned map. The unit of work for `georef` and `trace`.

**Trace.** A line's geometry as drawn from a sheet. A **candidate trace** is a machine
proposal nobody has reviewed yet.
