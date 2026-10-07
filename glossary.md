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

**Era-keyed vocabulary.** Search terms that depend on the period: `gas works` in 1887,
`pipe line` in 1937.

**Junction.** A node where segments meet. Recorded only where a source shows one (I-0007).

**Lane.** Which license, or which retrieval path, a piece of data came through. Recorded at
ingest.

**Map-date rule.** A line on a map published in year N proves existence by N, not
construction in N (I-0003).

**Segment.** A stretch of line between two nodes: a pipeline, a transmission line, a road, a
main.

**Sheet.** One scanned map. The unit of work for `georef` and `trace`.

**Trace.** A segment's geometry as drawn from a sheet. A **candidate trace** is a machine
proposal nobody has reviewed yet.
