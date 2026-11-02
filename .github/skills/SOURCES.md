# Skill sources and local adaptations

Research checked on 2026-11-02. Repository star counts below are approximate
discovery signals, not measured skill installs or evidence of output quality.
Prefer established, inspectable skills and official tool workflows over
marketplace copies with unclear provenance.

| Backend | Selected upstream | Discovery signal | License |
|---|---|---|---|
| Mermaid | [softaworks/agent-toolkit: mermaid-diagrams](https://github.com/softaworks/agent-toolkit/tree/3027f20f3181758385a1bb8c022d4041dfb4de84/skills/mermaid-diagrams) | ~2.6k repository stars; extensive per-type references | [MIT](./mermaid-diagrams/LICENSE) |
| Draw.io | [jgraph/drawio-mcp: Copilot drawio skill](https://github.com/jgraph/drawio-mcp/tree/30c232a35a924e193cd419013b3deb35536ec79b/plugins/copilot/skills/drawio) | Official publisher; ~5.7k stars | [Apache-2.0](./drawio/LICENSE) |
| Raw SVG | [Cocoon-AI/architecture-diagram-generator](https://github.com/Cocoon-AI/architecture-diagram-generator/tree/4b9087d55268c79a935105439dbcd37b630fc3f3/architecture-diagram) | ~7.5k stars; native SVG authoring rules | [MIT](./architecture-diagram/LICENSE) |
| Excalidraw | [axtonliu/axton-obsidian-visual-skills: excalidraw-diagram](https://github.com/axtonliu/axton-obsidian-visual-skills/tree/1265976d9746a84858b4b7b42fb86a215aa93de9/excalidraw-diagram) | ~3.7k stars; native Standard JSON and bundled schema | [MIT](./excalidraw-diagram/LICENSE) |
| Diagrams (Python) | [sammcj/agentic-coding: diagrams-as-code](https://github.com/sammcj/agentic-coding/tree/03ae89e44a89c601169f5fcdd7a34ab717917dd5/Skills_disabled/diagrams-as-code) | ~167 stars; focused library API and provider reference | [Apache-2.0](./diagrams-as-code/LICENSE) |
| Figma MCP | [Figma's official figma-generate-diagram guidance](https://github.com/figma/mcp-server-guide/tree/f3f264d9cbae059d9b96289ee33ce444b5c8e7e6/skills/figma-generate-diagram) | Official publisher; ~2.1k stars | No repository license declared; linked, **not copied** |

## What was reused

Five skill directories were installed with the `skills` CLI from the exact
revisions above, including their bundled references/assets, then moved into
`.github/skills`. The original technical bodies remain in their `SKILL.md`
files. Local changes add an overriding comparison-mode section and disable
automatic/slash invocation so the dispatcher loads each file explicitly.
The Python skill's Claude-specific `allowed-tools` field was removed; the
VS Code agent's tool allowlist is used instead.

[skills-lock.json](../../skills-lock.json) records the **upstream** revisions
and upstream content hashes, not the hashes of our adapted copies. It is
provenance, not a local integrity check. Do not run automatic skill updates or
restore over these adaptations: the installer currently targets `.agents/skills`,
and restoring would remove comparison restrictions or introduce duplicate skills.
Review an upstream diff, update the vendored files, and reapply the local
comparison contract deliberately.

The Python source is in upstream's `Skills_disabled` directory. It is reused as
a readable API guide, not presented as an officially supported or widely
installed skill. Its workflow was cross-checked against the
[Diagrams installation guide](https://diagrams.mingrammer.com/docs/getting-started/installation)
and [Diagram API guide](https://diagrams.mingrammer.com/docs/guides/diagram).

The raw SVG source normally emits HTML with an SVG inside it. Comparison mode
reuses its SVG palette, elements, spacing, and layering, but writes **standalone
SVG**, with no HTML dashboard, summary cards, external assets, or export UI.
This is a deliberate adaptation, not a claim that upstream natively outputs
only raw SVG.

The draw.io XML essentials reference was additionally bundled from
`shared/xml-essentials.md` at the pinned jgraph revision, with only ASCII
substitutions. Comparison mode selects native XML authoring to avoid an
interactive editor or a draw.io Desktop prerequisite. No repair pass is run.

The Figma skill is a small, independently written adapter because the official
guidance did not declare a redistribution license. It uses `generate_diagram`,
which accepts Mermaid and returns editable FigJam content and a link. It does
not pretend that `get_design_context` (a read/code tool) creates diagrams.
The ordinary layout is used; the optional architecture layout and hybrid
`use_figma` workflow are intentionally excluded.

## Comparison-specific deviations

The [comparison contract](./COMPARISON.md) overrides upstream recommendations
to research the subject, infer architecture, iterate on aesthetics, explain
choices, install tools, open applications, or propose improvements. It also
fixes output paths and filenames. Sources are retained alongside rendered
artifacts, and failures are recorded rather than hidden behind alternate tools.
These restrictions trade some upstream automation for consistent inputs.

## Supporting documentation

- [VS Code custom agents and tool allowlists](https://code.visualstudio.com/docs/agent-customization/custom-agents)
- [VS Code skill loading and directory structure](https://code.visualstudio.com/docs/agent-customization/agent-skills)
- [Figma MCP server guide](https://github.com/figma/mcp-server-guide)
- [Mermaid CLI](https://github.com/mermaid-js/mermaid-cli)
- [Diagrams library](https://diagrams.mingrammer.com/)

Popular alternatives were considered, including
[coleam00/excalidraw-diagram-skill](https://github.com/coleam00/excalidraw-diagram-skill)
(~5.2k stars, but no repository license declared) and
[yctimlin/mcp_excalidraw](https://github.com/yctimlin/mcp_excalidraw)
(~2.6k stars, MIT, but adds a live canvas/server workflow). The selected native
JSON skill avoids both redistribution uncertainty and another running service.
