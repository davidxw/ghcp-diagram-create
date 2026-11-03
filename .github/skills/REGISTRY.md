# Diagram skill registry

This is the single source of truth for the available diagram backends, selection
names, and execution order. [The dispatcher](../agents/create-diagrams.agent.md)
reads this file before resolving a selection. Every row participates in the
default selection and `Skills: all`; do not infer backends from directory scans,
the README, or installed tools. The dispatcher executes selected rows one at a
time and reports results in the order below.

## Registered backends

| Tool directory | Display name / aliases | Skill to load | Normal deliverables |
|---|---|---|---|
| `mermaid` | Mermaid | [mermaid-diagrams](./mermaid-diagrams/SKILL.md) | Mermaid source + rendered SVG or requested format |
| `drawio` | Draw.io | [drawio](./drawio/SKILL.md) | Editable draw.io file or MCP diagram + requested exports |
| `svg` | Architecture HTML+SVG, HTML+SVG, Raw SVG, `raw-svg` | [architecture-diagram](./architecture-diagram/SKILL.md) | HTML with inline SVG; standalone SVG or exports when requested |
| `excalidraw` | Excalidraw | [excalidraw-diagram](./excalidraw-diagram/SKILL.md) | Standard `.excalidraw` by default; Obsidian or Animated when explicitly requested |
| `diagrams-python` | Diagrams (Python), Python Diagrams, `diagrams` | [diagrams-as-code](./diagrams-as-code/SKILL.md) | Python source + rendered image in the chosen format |
| `excalidraw-cli` | Excalidraw-cli, Excalidraw CLI | [excalidraw-cli](./excalidraw-cli/SKILL.md) | Element JSON + CLI-generated Standard `.excalidraw`, plus requested exports |

These are native/default deliverables, not the complete success criteria.
[The comparison contract](./COMPARISON.md) also requires a validated local
PNG/JPG preview, `status.json`, requested exports, and inclusion in the HTML run
summary.

## Registry rules

- The tool directory is the canonical backend ID, its output subdirectory under
  `run_dir`, and its `status.json` `tool` value. Use a unique lowercase
  alphanumeric/hyphen name, not a path.
- Accept the complete tool directory, display name, comma-separated alias, or
  linked skill name, case-insensitively after trimming whitespace. Backticks are
  formatting, not part of a name. The linked name must match the skill directory
  and its frontmatter `name`. Match complete names, never substrings.
- Names must not resolve to different rows after normalization. `all` is reserved
  for selecting every row, not a backend name or alias. Repeated equivalent names
  for the same row select it once.
- Each row must link to an existing local `SKILL.md` beneath this directory.
  Keep backend-specific defaults and technical instructions in that skill;
  do not copy them into the dispatcher.
- `Excalidraw` selects only the direct-authoring row. `Excalidraw-cli` and
  `Excalidraw CLI` select only the CLI row. `Skills: excalidraw, excalidraw-cli`
  selects both independently.
- The legacy `svg` directory and Raw SVG aliases select Architecture HTML+SVG,
  not a standalone SVG output request. Formats and modes are specified separately.

For additions, use [add-diagrams-skill](../agents/add-diagrams-skill.agent.md).
Update this registry, the skill, and its documentation/provenance together.
