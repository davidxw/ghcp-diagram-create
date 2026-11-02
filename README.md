# ghcp-diagram-create

Compare agent-driven diagram tools in GitHub Copilot using the same unchanged
prompt. Five established upstream skills are vendored and adapted; the Figma
adapter follows its official MCP workflow. See the
[research, sources, licenses, and adaptations](.github/skills/SOURCES.md).

## Tools and prerequisites

| Tool | Repository skill | Required setup | Successful output |
|---|---|---|---|
| Mermaid | [mermaid-diagrams](.github/skills/mermaid-diagrams/SKILL.md) | Node.js and Mermaid CLI (`mmdc`) on PATH, with its browser dependencies available | Mermaid source + SVG |
| Draw.io | [drawio](.github/skills/drawio/SKILL.md) | None beyond local XML parsing; uses native XML authoring | Editable `.drawio` |
| Raw SVG | [architecture-diagram](.github/skills/architecture-diagram/SKILL.md) | None beyond local XML parsing; adapted to standalone SVG | Standalone `.svg` |
| Excalidraw | [excalidraw-diagram](.github/skills/excalidraw-diagram/SKILL.md) | None beyond local JSON parsing; uses Standard mode | Editable `.excalidraw` |
| Diagrams (Python) | [diagrams-as-code](.github/skills/diagrams-as-code/SKILL.md) | Python, `diagrams`, and Graphviz (`dot`) on PATH | Python source + SVG |
| Figma MCP | [figma-diagram](.github/skills/figma-diagram/SKILL.md) | Official remote Figma MCP server, authenticated, with `generate_diagram` enabled | Mermaid source + FigJam link + tool response |

Install prerequisites yourself before running the agent. For example, on Windows:

```powershell
npm install --global @mermaid-js/mermaid-cli
winget install Graphviz.Graphviz
python -m pip install diagrams
```

Graphviz may require a new terminal/VS Code session to refresh PATH.
The agent never installs packages, starts servers, or repairs the environment.
PowerShell provides local XML/JSON parsing on Windows.

[The workspace MCP configuration](.vscode/mcp.json) defines `figma` at
`https://mcp.figma.com/mcp`. Use VS Code's MCP controls to enable it and complete
Figma authentication before invoking the agent. Its tool allowlist names
`figma/generate_diagram`; renaming the server requires updating that entry.
Figma sends the diagram content to the Figma service and creates cloud FigJam
content. It does **not** export a local image in this workflow.

## Usage

1. Open this repository in VS Code with GitHub Copilot.
2. Select [create-diagrams](.github/agents/create-diagrams.agent.md) in the agent
   picker. If it is not shown, reload the window and check Chat Customizations.
3. Submit a self-contained prompt with explicit entities, relationships, and
   diagram type. For example:

   > Create a left-to-right flowchart with three nodes labelled Client, API,
   > and Database. Connect Client to API with an arrow labelled HTTPS.
   > Connect API to Database with an arrow labelled SQL.

By default, the agent loads all six local skills. To run a smaller comparison,
explicitly select one or more tools in the prompt:

```text
Skills: mermaid, draw.io

Create a left-to-right flowchart with three nodes labelled Client, API,
and Database. Connect Client to API with an arrow labelled HTTPS.
Connect API to Database with an arrow labelled SQL.
```

You can also write "Use only Mermaid and Excalidraw" or "Use all except Figma"
as an explicit selection instruction. A tool name mentioned only as diagram
content does not change the selection. Names are case-insensitive; the tool
names above, directory names in the output layout below, and linked skill names
are accepted. Duplicate names are run once. `Skills: all` selects all six.
Do not combine `all` with individual names in a `Skills:` list. An empty,
unknown, ambiguous, or conflicting selection is reported as `blocked` before
a run is created; it is not silently ignored.

The agent loads only the selected skills, in the order shown above. It saves and
passes the entire prompt unchanged, treating selection instructions as routing
metadata rather than diagram content. It does not summarize or enrich the prompt,
inspect the repository, reuse another tool's diagram, invent components, rank
outputs, or offer improvements. Only backend-specific syntax, layout, and
serialization/rendering checks are permitted.

Missing prerequisites, ambiguity, and unsupported features produce `blocked`;
execution or validation errors produce `failed`. The remaining selected skills
still run. There are no substitutions, retries, or silent success fallbacks.
Not every tool supports every diagram type: Python Diagrams primarily targets
architecture/infrastructure, while Figma supports a subset of Mermaid types.

## Output location

Every invocation creates a new directory under repository-root `diagrams/`:

```text
diagrams/<inferred name>-<random-suffix>/
  prompt.txt
  mermaid/          diagram.mmd, diagram.svg, status.json
  drawio/           diagram.drawio, status.json
  svg/              diagram.svg, status.json
  excalidraw/       diagram.excalidraw, status.json
  diagrams-python/ diagram.py, diagram.svg, status.json
  figma/            diagram.mmd, diagram.url, response.txt, status.json
```

The layout above shows all possible backends. Only selected backends get
directories and status files, even if blocked or failed; unselected backends
are not loaded or checked. Error diagnostics go in that backend's `error.txt`;
partial files are retained but are not reported as successful artifacts. The
final response is only a status/artifact table with one row per selected skill.
Sources and native editable files are retained; not every backend has a local
image preview.

The `.url` file is a local shortcut to the actual FigJam diagram, not a local
copy of that diagram. The returned tool response is saved unchanged.

## Restrictions and verification

The [shared comparison contract](.github/skills/COMPARISON.md) overrides upstream
defaults that would add research, explanations, alternate formats, or follow-up
work. The agent exposes only file reading/editing, execution required by the
skills, and Figma's diagram-generation tool; subagents are disabled.

**Skills are instructions, not a security sandbox.** VS Code's tool allowlist
limits available capabilities, but does not enforce per-skill permissions or
filesystem boundaries. The skill-only and no-inference rules are instructional,
as are the output-directory restrictions. Review tool approvals accordingly.

To check behavior in your own Copilot environment:

- Run the first three-node prompt above without a selection; expect six status
  files and the same three labels/two relationships wherever the backend
  succeeds.
- Use `Skills: mermaid, draw.io`; expect only two status files and two table
  rows, with no Figma call or directories for unselected skills.
- Select one skill by its linked skill name; expect one status file and one row.
- Use `Skills: mermaid, unknown`; expect `blocked` and no run directory.
- Without a renderer or authenticated Figma server, expect `blocked` for that
  selected backend, no installation/authentication attempt, and continued
  execution of the other selected skills.
- Request a class diagram with Figma selected; it must report `blocked`, not
  generate a flowchart.
- Use an ambiguous prompt; expect `blocked` rather than invented components.
- Confirm each run has a new directory and the original prompt is unchanged.

Customizations can be checked statically, but successful rendering and strict
instruction adherence must also be tested in the intended Copilot harness with
its actual tool permissions, installed renderers, and authenticated MCP server.