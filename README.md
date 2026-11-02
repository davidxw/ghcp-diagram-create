# Create Diagrams with GitHub Copilot

This repo can be used to compare different approaches and skills for creating diagrams with GitHub Copilot or other AI coding agents.  Diagram approaches are codified in skills, and an agent runs the same prompt against all these skills and presents the output.

## Quickstart

1. Review the tool list and pre-requites for each tool
2. Use the `create-diagrams` agent with a description of your diagram to generate outputs for all selected tools and skills.
3. Use the `add-diagrams-skill` agent to add new diagram approaches as skills to the repository.


Each selected skill uses its normal authoring, layout, rendering, and refinement workflow. Upstream (source)
skills and local workflow adapters provide the approaches listed in
[the backend registry](.github/skills/REGISTRY.md). 

See the
[research, sources, licenses, and adaptations](.github/skills/SOURCES.md) for the full source list.

## Tools and prerequisites

| Tool | Repository skill | Approach | Required setup | Successful output |
|---|---|---|---|---|
| Mermaid | [mermaid-diagrams](.github/skills/mermaid-diagrams/SKILL.md) | Mermaid code + local renderer | Node.js + Mermaid CLI (`mmdc`) with its browser dependencies, or another available local Mermaid renderer | Mermaid source + SVG by default, or requested rendered format |
| Draw.io | [drawio](.github/skills/drawio/SKILL.md) | XML/Mermaid + CLI or MCP | **draw.io Desktop recommended** for Mermaid conversion, automatic layout/routing, and SVG/PNG/PDF export; Node.js 18+ for the bundled shape lookup, XML repair, and URL helpers. An available draw.io MCP/editor route is also supported | Editable `.drawio` or MCP diagram, plus PNG/JPG preview and requested exports; XML-only authoring also needs a compatible preview renderer |
| Architecture HTML+SVG | [architecture-diagram](.github/skills/architecture-diagram/SKILL.md) | Native HTML/SVG generation | Modern browser for preview; browser tooling for automated inspection/export. Internet access for the template's Google Font and CDN export libraries | Styled HTML with inline SVG and export toolbar; standalone SVG or PNG/PDF when requested |
| Excalidraw | [excalidraw-diagram](.github/skills/excalidraw-diagram/SKILL.md) | Native JSON generation | Local JSON parsing for native authoring; compatible Excalidraw viewer/exporter or browser workflow required for the PNG/JPG preview. Obsidian + Excalidraw plugin only for explicitly requested Obsidian output | Standard `.excalidraw` by default; Obsidian `.md` or Animated `.excalidraw` when explicitly requested; Sloppiness defaults to **Architect** (leftmost, `roughness: 0`) for comparison with other tools |
| Diagrams (Python) | [diagrams-as-code](.github/skills/diagrams-as-code/SKILL.md) | Python code + Graphviz | Python environment containing `diagrams` + Graphviz (`dot`); network access only when custom icons are needed | Python source + PNG by default, or SVG/another supported requested format |
| Figma MCP | [figma-diagram](.github/skills/figma-diagram/SKILL.md) | External MCP | Official remote Figma MCP server, authenticated; `generate_diagram` for the base route; supported image export/screenshot or browser access to the actual board for the preview; optional FigJam editing tools for extensions | Authoring source + FigJam link + tool responses + local PNG/JPG preview; requested annotations/exports when supported |
| Excalidraw-cli | [excalidraw-cli](.github/skills/excalidraw-cli/SKILL.md) | JSON + CLI authoring | Node.js 18+ and installed `excalidraw-cli`; compatible viewer/exporter or browser workflow required for the PNG/JPG preview | Element JSON + CLI-generated Standard `.excalidraw`; defaults to Sloppiness **Architect** (`roughness: 0`); image rendering requires a separate compatible viewer/exporter |

Every successful backend retains a PNG or JPG view of its final diagram in
addition to the native deliverables above, even when another format is requested.
An existing final PNG/JPG export is reused rather than duplicated. Each run also
produces an offline-readable HTML summary with these previews and artifact links.

Install prerequisites before running the agent. For example, on Windows:

```powershell
npm install --global @mermaid-js/mermaid-cli
npm install --global excalidraw-cli
winget install Graphviz.Graphviz
python -m pip install diagrams
```

- Install a current [draw.io Desktop](https://github.com/jgraph/drawio-desktop/releases)
  build for its full workflow. Older builds may lack Mermaid input or some layout
  flags; the skill checks capabilities and uses supported alternatives.
  Windows installations need not be on PATH: the skill also checks normal
  installation locations and registry entries.
- The optional [Draw.io Integration extension](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio)
  provides an editable preview inside VS Code.
- Only for explicitly requested Excalidraw Obsidian output, install [Obsidian](https://obsidian.md/) and its
  [Excalidraw community plugin](https://obsidian.md/plugins?id=obsidian-excalidraw-plugin).
  Standard JSON authoring does not need these. The required raster preview and
  any image or animation exports need a compatible viewer/exporter or browser
  workflow; animation metadata alone is not a video export.
- Excalidraw-cli is a separate CLI-assisted authoring approach, not the direct
  Excalidraw skill and not an automatic layout or image-rendering engine. It
  supports label shorthand and local checkpoints, kept inside the run directory.
  Its `export` command uploads to Excalidraw for a sharing link, **not** PNG/SVG;
  that upload is opt-in and is never part of ordinary local diagram creation.
- Enable your host's browser tools, or provide an already installed local browser
  automation tool, for visual inspection and automated HTML exports. A browser
  executable alone does not necessarily give the agent browser automation.
  Architecture HTML remains readable offline with font fallbacks; its export
  toolbar needs the pinned CDN libraries to load.

Graphviz may require a new terminal/VS Code session to refresh PATH. The selected
Python environment must contain `diagrams` and be able to find `dot`.
PowerShell provides local XML/JSON parsing on Windows. The agent uses preinstalled
tools: it does not install dependencies, download container images, reconfigure
the machine, or authenticate accounts. It may start a temporary local preview
server for inspection and stops task-owned preview processes before returning.

[The repository MCP configuration](.mcp.json) names the official Figma endpoint
`figma` at `https://mcp.figma.com/mcp`. If your host does not read this configuration
format, add that HTTP server through its MCP controls (for example, VS Code's
**MCP: Add Server**) and complete authentication before running the agent.
Enable any additional FigJam inspection/editing tools needed for annotations;
their availability depends on the server and account. A draw.io MCP integration
is optional and must likewise be configured before a run.

Figma sends the diagram content to its service and creates cloud FigJam content.
Local CLI rendering needs no diagram upload. Public technical documentation and
icon lookups are allowed, but private diagram content is not sent to additional
hosted editors/renderers without permission.

## Usage

1. Open this repository in VS Code with GitHub Copilot.
2. Select [create-diagrams](.github/agents/create-diagrams.agent.md) in the agent
   picker. If it is not shown, reload the window and check Chat Customizations.
3. Submit a self-contained prompt with explicit entities, relationships, and
   diagram type. For example:

   > Create a left-to-right flowchart with three nodes labelled Client, API,
   > and Database. Connect Client to API with an arrow labelled HTTPS.
   > Connect API to Database with an arrow labelled SQL.

By default, the agent loads all registered local skills. To run a smaller comparison,
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
are accepted. Duplicate names are run once. `Skills: all` selects every registered backend.
Do not combine `all` with individual names in a `Skills:` list. An empty,
unknown, ambiguous, or conflicting selection is reported as `blocked` before
a run is created; it is not silently ignored.

`Excalidraw` selects the direct-authoring skill; `Excalidraw-cli` or
`Excalidraw CLI` selects the new CLI approach. They are distinct selections.
Use `Skills: excalidraw, excalidraw-cli` to compare just these two.

The architecture skill retains the legacy `svg` directory and accepts the older
names `Raw SVG` and `raw-svg`. Those names select the skill, not an output format;
request "standalone SVG" explicitly if you want that instead of its normal HTML.

The agent loads only the selected skills, in
[registry order](.github/skills/REGISTRY.md). It saves and
passes the entire prompt unchanged, treating selection instructions as routing
metadata rather than diagram content. Each skill authors independently from the
same input, not another tool's diagram. If clarification is necessary, the answer
is saved separately and shared with all selected skills. Files/documents explicitly
supplied as diagram input may be read; unrelated project exploration is excluded.

Normal diagram work includes technical references, icon discovery, format
conversion, automatic layout/routing, editor/browser previews, rendering,
repair, and refinement of concrete defects. The agent stops when the diagram
meets the request and its checks pass. It does not redesign the architecture,
invent components, generate domain reports or rankings, or do follow-up work.
The HTML run summary is required output bookkeeping, not a domain report.

A supported alternative within the same backend is allowed, such as native
draw.io XML without Desktop, with any reduced capabilities disclosed. Missing
required setup or unsupported features produce `blocked`; unresolved execution
or validation errors produce `failed`. Recoverable errors can be fixed and
retried. Remaining selected skills still run; none is silently replaced with
another backend. A native file or cloud link without the required PNG/JPG view
is not `created`: unavailable preview tooling makes that backend `blocked`.
Not every tool supports every diagram type: Python Diagrams
primarily targets architecture, while Figma's generator supports only a subset
of Mermaid and may need optional FigJam tools for annotations.

## More complex sample prompts

### Grouped application architecture

```text
Create a left-to-right architecture diagram for an online store.

Organize these twelve components into four labelled groups:
- Clients: Browser, Mobile App.
- Application: API Gateway, Identity Provider, Product Service, Order Service.
- Data: Cache, Product Database, Order Database.
- Asynchronous Processing: Event Queue, Fulfillment Worker, Email Service.

Draw these directed connections with the specified labels:
- Browser -> API Gateway: HTTPS.
- Mobile App -> API Gateway: HTTPS.
- API Gateway -> Identity Provider: Token validation.
- API Gateway -> Product Service: Product queries.
- API Gateway -> Order Service: Order requests.
- Product Service -> Cache: Cache lookup.
- Product Service -> Product Database: Cache miss.
- Order Service -> Order Database: Order transactions.
- Order Service -> Event Queue: OrderCreated.
- Event Queue -> Fulfillment Worker: Consume event.
- Fulfillment Worker -> Order Database: Fulfillment status.
- Fulfillment Worker -> Email Service: Order confirmation.

Use distinct shapes for databases and the queue where supported. Keep group
boundaries, component labels, and connection labels readable, and minimize
crossing connectors. Do not add components or relationships.
```

### Order workflow with bounded retries

```text
Create a top-to-bottom order-processing flowchart. Use decision diamonds,
label every decision branch, and show explicit Accepted and Rejected end states.

Include this validation and inventory flow:
- Start -> Validate order -> Valid?
- Valid? No -> Reject order -> Rejected.
- Valid? Yes -> Reserve stock -> Stock available?
- Stock available? No -> Notify unavailable -> Rejected.
- Stock available? Yes -> Charge payment -> Payment result?

Payment result? has three branches:
- Approved -> Confirm order -> Publish OrderCreated -> Accepted.
- Declined -> Release stock -> Notify payment failure -> Rejected.
- Retryable error -> Attempts remaining?

Count the initial Charge payment execution as attempt one, with at most three
total payment attempts. Show this limit beside Attempts remaining?:
- Yes -> Wait 5 seconds -> Charge payment.
- No -> Release stock -> Notify payment failure -> Rejected.

Reuse the same Charge payment, Release stock, Notify payment failure, and
Rejected nodes rather than duplicating them. Make the retry loop visually
distinct from the main success path. Do not add steps or branches.
```

## Output location

Every invocation creates a new directory under repository-root `diagrams/`:

```text
diagrams/<inferred name>-<random-suffix>/
  index.html         (run summary, including the exact original prompt)
  clarifications.txt  (only if clarification was needed)
  mermaid/          diagram.mmd, diagram.svg (or requested format), diagram.png, status.json
  drawio/           diagram.drawio (or MCP source/link), diagram.png, requested exports, status.json
  svg/              diagram.html (or requested standalone SVG), diagram.png, exports, status.json
  excalidraw/       diagram.md / diagram.excalidraw / diagram.animate.excalidraw, diagram.png, status.json
  diagrams-python/ diagram.py, diagram.png (or chosen format), status.json
  figma/            authoring source, diagram.url, response.txt, diagram.png, status.json
  excalidraw-cli/   elements.json, diagram.excalidraw, diagram.png, status.json
```

The layout above illustrates normal outputs, not a requirement to export every
format. PNG/JPG previews are mandatory; a final JPG or differently named PNG
export can replace the illustrated `diagram.png`. Python output in a non-raster
format must also include the preview. Only selected backends get
directories and status files, even if blocked or failed; unselected backends
are not loaded or checked. Error diagnostics go in that backend's `error.txt`;
partial files are retained but are not reported as successful artifacts. The
final response links the HTML summary and run directory, followed by a
status/artifact table with one row per selected skill.
Sources and native editable files are retained, even when exporting an image.
A `created` result must include the chosen native deliverable, a decoded and
inspected PNG/JPG preview, and every explicitly requested export. Final previews
are kept during cleanup. The `status.json` fields include `preview` (a local
artifact path, or `null` for blocked/failed results) and `tokens_used` (an exact
backend-attributable integer, or `null` when unavailable). Its reason field
discloses material limitations instead of claiming checks that were not performed.

### HTML run summary

`index.html` replaces `prompt.txt` for new runs; existing runs are not migrated.
It is created at the start to preserve the original prompt, then completed using
the selected backends' final status files. It contains:

- **General:** the entire unchanged prompt, including selection instructions,
  plus the actual model used when the host exposes it.
- **For each selected skill:** status, saved preview image, blocked/failed reason
  or material caveats, links to generated artifacts (including the actual cloud
  diagram where applicable), and tokens used when reliably available.

Unknown model or token metadata is labelled **Not available**, not guessed,
estimated, or replaced with zero. Shared/session token totals are not allocated
across skills. Blocked/failed results have an explicit unavailable-preview label,
not a broken image or links claiming partial output as successful artifacts.
The summary uses escaped prompt text, relative local artifact/image links, and
inline styling with no external dependencies. It opens locally without uploading
the prompt or artifacts. Browser inspection checks the summary where available;
any unavailable summary-browser verification is disclosed in the final response.

The Figma `.url` file is a local shortcut, not a local copy of the editable board.
Tool responses are preserved unchanged. Refinements reuse the actual board/file
key where supported rather than creating unnecessary cloud drafts.

## Adding a new diagram approach

Select [add-diagrams-skill](.github/agents/add-diagrams-skill.agent.md) in the agent
picker and provide the drawing tool/approach plus an upstream URL or local skill.
For example:

```text
Add a new diagram approach from <source URL or local skill path>.
Use <display name> as its name and <backend-id> as its selection/output ID.
```

The maintenance agent contains the required/conditional file checklist and
guidance for adapting upstream skills, preserving licenses, documenting setup,
and checking previews/statuses. It updates
[the registry](.github/skills/REGISTRY.md), the skill, README, and provenance
together. The registry owns the available backend list, names/aliases, and
execution order; adding a backend does not normally require a dispatcher edit.
New skills must retain their normal diagram workflow and follow
[the shared contract](.github/skills/COMPARISON.md), including the required
PNG/JPG preview and HTML run-summary integration.

