# Open questions

Grouped by whose call it is.

## The research partner's, or joint

- **The dataset license.** MIT covers code; it does not answer for data, and the sources
  (OSM, GEM, US public domain) constrain what is possible.
- **Rights on the scans.** Holding institutions often claim rights over their digital images,
  and some mid-century maps may still be in copyright. Serving a georeferenced copy publicly
  is a rights decision before it is a technical one.
- **Where the profile URI lives.** The current value is a placeholder; nothing resolves
  there.
- **Identity calls.** How to decide when two observations are the same pipeline.
- **Who pays for inference at scale**, once there is a costed estimate.
- **Hosting after the research phase.** The research partner, or somewhere else.

## Ours, to settle by building

- **How `store` models disagreement and identity.** See `store` in components.
- **Where the audit trail lives**, given QGIS can write to PostGIS directly.
- **Whether the self-hosted Allmaps editor is good enough** as the georeferencing review
  tool. Try it on a real sheet.
- **QGIS or web for trace review.** The deck said QGIS; the plan says web.
- **What the group's spreadsheets look like.** Need a sample.
- **How exa fallback citations stay resolvable** after the URL rots.
- **When the pipeline runtime becomes its own library.**
