# CLAUDE.md

This is the Lineage controlplane: planning documents, no code. Read `README.md` first, then
`components.md`.

- Describe *what* each component does and its inputs and outputs. *How* belongs in the
  component's own repo.
- One meaning, one home. A component's description lives in `components.md`; its repo's
  README links here rather than copying it.
- Decisions go in `decisions.md`. `L-` entries were made for Lineage; `I-` entries are
  inherited from Legend and advisory. Reopening one is fine; say what the earlier reasoning
  was.
- Use the words in `glossary.md`.
- No em-dashes or en-dashes as separators. Use a spaced hyphen, a semicolon or a new
  sentence.
- This repo is public, and so is participation in Lineage (L-0010). Name people and the
  research partner where it helps; prefer GitHub handles for org members. Don't name
  Tight Line's other clients, or specific maps or sheets: say "a real sheet", "a spike
  sheet".
- These repos are software, not deployment configuration. No hostnames, cluster names,
  internal repos or environment-specific settings.
