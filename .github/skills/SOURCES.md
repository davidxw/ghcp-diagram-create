# Skill sources and local adaptations

Research checked on 2026-11-02. Repository star counts below are approximate
discovery signals, not measured skill installs or evidence of output quality.
Prefer established, inspectable skills and official tool workflows over
marketplace copies with unclear provenance.

| Backend | Selected upstream | Discovery signal | License |
|---|---|---|---|
| Mermaid | [softaworks/agent-toolkit: mermaid-diagrams](https://github.com/softaworks/agent-toolkit/tree/3027f20f3181758385a1bb8c022d4041dfb4de84/skills/mermaid-diagrams) | ~2.6k repository stars; extensive per-type references | [MIT](./mermaid-diagrams/LICENSE) |
| Draw.io | [jgraph/drawio-mcp: Copilot drawio skill](https://github.com/jgraph/drawio-mcp/tree/30c232a35a924e193cd419013b3deb35536ec79b/plugins/copilot/skills/drawio) | Official publisher; ~5.7k stars | [Apache-2.0](./drawio/LICENSE) |
| Architecture HTML+SVG | [Cocoon-AI/architecture-diagram-generator](https://github.com/Cocoon-AI/architecture-diagram-generator/tree/4b9087d55268c79a935105439dbcd37b630fc3f3/architecture-diagram) | ~7.5k stars; HTML template and inline SVG design rules | [MIT](./architecture-diagram/LICENSE) |
| Excalidraw | [axtonliu/axton-obsidian-visual-skills: excalidraw-diagram](https://github.com/axtonliu/axton-obsidian-visual-skills/tree/1265976d9746a84858b4b7b42fb86a215aa93de9/excalidraw-diagram) | ~3.7k stars; native Standard JSON and bundled schema | [MIT](./excalidraw-diagram/LICENSE) |
| Diagrams (Python) | [sammcj/agentic-coding: diagrams-as-code](https://github.com/sammcj/agentic-coding/tree/03ae89e44a89c601169f5fcdd7a34ab717917dd5/Skills_disabled/diagrams-as-code) | ~167 stars; focused library API and provider reference | [Apache-2.0](./diagrams-as-code/LICENSE) |
| Figma MCP | [Figma's official figma-generate-diagram guidance](https://github.com/figma/mcp-server-guide/tree/f3f264d9cbae059d9b96289ee33ce444b5c8e7e6/skills/figma-generate-diagram) | Official publisher; ~2.1k stars | No repository license declared; linked, **not copied** |

## What was reused

Five skill directories were installed with the `skills` CLI from the exact
revisions above, including their bundled references/assets, then moved into
`.github/skills`. The upstream technical guidance is retained in the `SKILL.md`
files, with the Excalidraw English adaptation noted below. Local workflow
sections add output locations, validation, and scope boundaries without
disabling the normal authoring/layout/rendering/refinement workflows.
Automatic/slash invocation is disabled so the dispatcher loads selected files
explicitly.
The Python skill's Claude-specific `allowed-tools` field was removed; the
VS Code agent's tool allowlist is used instead.

[skills-lock.json](../../skills-lock.json) records the **upstream** revisions
and upstream content hashes, not the hashes of our adapted copies. It is
provenance, not a local integrity check. Do not run automatic skill updates or
restore over these adaptations: the installer currently targets `.agents/skills`,
and restoring would remove local workflow adaptations or introduce duplicate skills.
Review an upstream diff, update the vendored files, and reapply the local
scope and bookkeeping contract deliberately.

The Python source is in upstream's `Skills_disabled` directory. It is reused as
a readable API guide, not presented as an officially supported or widely
installed skill. Its workflow was cross-checked against the
[Diagrams installation guide](https://diagrams.mingrammer.com/docs/getting-started/installation)
and [Diagram API guide](https://diagrams.mingrammer.com/docs/guides/diagram).

The architecture skill now uses its normal HTML template with inline SVG,
typography, styling, and export toolbar. The older `svg` output directory and
Raw SVG aliases are retained for selection compatibility; standalone SVG is an
explicit output option rather than a forced replacement for the HTML workflow.
Template examples/placeholders and unused cards are removed; any annotations
or summary cards must stay grounded in the supplied diagram content.

The draw.io XML essentials reference was additionally bundled from
`shared/xml-essentials.md` at the pinned jgraph revision, with only ASCII
substitutions. Draw.io now follows its normal Mermaid/Desktop, native XML, or
available MCP/editor route. Shape lookup, XML repair, supported automatic layout
and routing, previews, troubleshooting, and requested exports are allowed.
Editable local sources are retained rather than deleted after image export.
Desktop is recommended for the full workflow; XML-only output remains an
explicitly disclosed option when Desktop is absent and it meets the request.

The Excalidraw skill's Chinese instructions, trigger phrases, sample text, and
filename examples were translated into English locally, including an English
filename preference. The upstream attribution and MIT license are retained.
All three native modes remain available. Standard `.excalidraw` is the local
default instead of upstream's Obsidian default; Obsidian and Animated modes are
used only when explicitly requested, so normal output has no Obsidian dependency.
Labels are preserved using JSON
escaping rather than the upstream punctuation substitutions. Obsidian wrapper
examples use nested Markdown fences correctly, and binding guidance allows
actual bound-element lists and valid editor-generated metadata.
Its element templates and bundled schema examples default to `roughness: 0`,
and its saved editor state uses `appState.currentItemRoughness: 0`, unless the
prompt explicitly requests different sloppiness. This is the leftmost
**Architect** option under **Sloppiness**, selected for clean-stroke comparison
with other tools rather than Excalidraw's usual sketchy default. The mapping is
defined in Excalidraw's [roughness constants](https://github.com/excalidraw/excalidraw/blob/master/packages/common/src/constants.ts)
and [sloppiness control](https://github.com/excalidraw/excalidraw/blob/master/packages/excalidraw/actions/actionProperties.tsx).

The Figma skill remains an independently written adapter because the official
guidance did not declare a redistribution license; its guides are linked, not
copied. It follows the official generator and type-specific guidance, including
architecture layout when compatible with the supplied graph and hybrid FigJam
extensions when relevant tools are available. Inspection, screenshots, supported
refinement, and requested annotations/exports are allowed. A read/design-context
tool is not treated as a diagram-creation API. Refinements reuse the actual
returned file key and preserve source/scripts and tool responses.

## Scope-only workflow adaptations

The [shared contract](./COMPARISON.md) keeps identical input, independent backend
authoring, per-run output directories, honest statuses, and the requirement not
to invent facts or do unrelated work. It allows technical documentation/icon
lookup, each skill's native output modes, conversion, automatic layout/routing,
browser/editor inspection, temporary preview servers, repair, and refinement.
There is no one-attempt limit or prohibition on normal diagram exports.

As requested, tools must be preinstalled and services authenticated before a
run; upstream install snippets and dependency-resolving launch commands are
setup guidance, not authorization to install packages during diagram creation.
Only explicitly supplied content sources are investigated. Unrelated domain
research, architecture redesign, project changes, reports, recommendations,
and optional follow-up work remain excluded. Temporary preview processes are
stopped at completion. Sources and successful final outputs are retained, and
unsupported requests or reduced capabilities are reported rather than hidden.

The dispatcher no longer hard-codes a narrow tool allowlist that would block
normal browser/MCP workflows. It uses host-available tools within these scope
rules; subagents are still disabled. These are instructions, not sandbox
enforcement, and do not bypass the host's approvals or service availability.

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
