# Create Diagrams with GitHub Copilot

Evaluate diagram-generation skills by running the same prompt through different
tools, then reviewing their diagrams side by side. Use GitHub Copilot in VS Code
or Copilot CLI.

## Quick Start

1. Open this repository in VS Code with GitHub Copilot.
2. Set up the tools you want to evaluate using the prerequisites below.
3. Select [create-diagrams](.github/agents/create-diagrams.agent.md) in the
   Copilot Chat agent picker and submit:

   ```text
   Create a left-to-right flowchart with three nodes labelled Client, API,
   and Database. Connect Client to API with an arrow labelled HTTPS.
   Connect API to Database with an arrow labelled SQL.
   ```

4. Open the generated `index.html` from the agent's response to review the
   diagrams and download or edit their sources.

If an agent is missing from the picker, reload the window and check Chat
Customizations.

## Tools and prerequisites

Install or configure prerequisites before starting an evaluation. You only need
setup for the tools you select; the agents do not install dependencies or
authenticate services for you.

| Tool / skill | Approach | Prerequisites |
|---|---|---|
| [Mermaid](.github/skills/mermaid-diagrams/SKILL.md) | Mermaid code + local renderer | Node.js and Mermaid CLI (`mmdc`) with its browser dependencies, or another local Mermaid renderer |
| [Draw.io](.github/skills/drawio/SKILL.md) | XML/Mermaid + Desktop or MCP | [draw.io Desktop](https://github.com/jgraph/drawio-desktop/releases) recommended; Node.js 18+ for bundled helpers. A configured MCP/editor route with preview/export support is also supported |
| [Architecture HTML+SVG](.github/skills/architecture-diagram/SKILL.md) | Native HTML with inline SVG | Modern browser and browser tooling for inspection/export; internet access for optional fonts and CDN export libraries |
| [Excalidraw](.github/skills/excalidraw-diagram/SKILL.md) | Native JSON authoring | Compatible Excalidraw viewer/exporter or browser workflow for image previews |
| [Diagrams (Python)](.github/skills/diagrams-as-code/SKILL.md) | Python code + Graphviz | Python environment containing `diagrams`, plus Graphviz (`dot`) |
| [Figma MCP](.github/skills/figma-diagram/SKILL.md) | FigJam through MCP | Authenticated official Figma MCP server with `generate_diagram`, plus image export or browser access for previews |
| [Excalidraw-cli](.github/skills/excalidraw-cli/SKILL.md) | JSON + CLI authoring | Node.js 18+ and `excalidraw-cli`; a separate compatible viewer/exporter is needed for image previews |

Example installations on Windows; choose only what you need:

```powershell
npm install --global @mermaid-js/mermaid-cli
npm install --global excalidraw-cli
winget install Graphviz.Graphviz
python -m pip install diagrams
```

After installing Graphviz, restart your terminal or VS Code if needed so `dot`
is on PATH. Enable your host's browser tools for visual inspection and exports.

The [MCP configuration](.mcp.json) includes the official Figma endpoint. If your
host does not load it, add the server through its MCP controls and authenticate
before running. Figma creates cloud diagrams and sends diagram content to its
service; local rendering does not require a diagram upload.

## Usage

Choose an execution mode, then submit a prompt describing the diagram type,
direction, nodes, and labelled connections. Each selected skill receives the same
input. Request a particular export format in the prompt if needed.

### Single-stream execution

Select the [create-diagrams](.github/agents/create-diagrams.agent.md) agent to run selected
skills one at a time. This is the simplest way to start an evaluation.

### Parallel execution

Select [create-diagrams-parallel](.github/agents/create-diagrams-parallel.agent.md)
to run a worker for each selected skill. In VS Code, enable **Run Subagent** in
**Configure Tools**. In Copilot CLI, use fleet mode:

```powershell
copilot --agent create-diagrams-parallel --fleet -i "Create a flowchart: Client -> API -> Database."
```

Parallel execution requires host subagent support. Actual concurrency depends
on the host and available tools; the agent reports any sequential fallback.
Subagents can increase model usage.

### Selecting tools

Both modes use all registered tools by default. Add a selection to your prompt
to evaluate a subset:

```text
Skills: mermaid, drawio, excalidraw
```

You can also use `Skills: all` or write "Use all except Figma". Do not combine
`all` with individual names. Names are case-insensitive; see the
[registry](.github/skills/REGISTRY.md) for accepted names and aliases.
`Excalidraw` and `Excalidraw-cli` are separate approaches.

### Outputs

Each evaluation creates a fresh folder under `diagrams/<name>-<random-suffix>/`
with:

- **`index.html`:** the prompt, diagram previews, artifact links, and each tool's
  status. Model and token usage are shown when available.
- **One folder per selected tool:** editable/native sources, a PNG/JPG preview,
  requested exports, and status or error details. Figma outputs include a link
  to the editable cloud board.

Open the summary locally to review the results. A tool is reported as `created`,
`blocked` when setup or capabilities are missing, or `failed` when an error
remains unresolved. Other selected tools still run if one cannot complete.

## Sample inputs

Use the Quick Start prompt for a simple labelled flowchart. These examples
exercise grouping, branching, and connector routing.

### Grouped application architecture

```text
Create a left-to-right architecture diagram for an online store.

Group these components:
- Clients: Browser, Mobile App.
- Application: API Gateway, Product Service, Order Service.
- Data: Product Database, Order Database.
- Asynchronous Processing: Event Queue, Fulfillment Worker.

Draw these labelled arrows:
- Browser -> API Gateway: HTTPS.
- Mobile App -> API Gateway: HTTPS.
- API Gateway -> Product Service: Product queries.
- API Gateway -> Order Service: Order requests.
- Product Service -> Product Database: Product lookup.
- Order Service -> Order Database: Order transactions.
- Order Service -> Event Queue: OrderCreated.
- Event Queue -> Fulfillment Worker: Consume event.
- Fulfillment Worker -> Order Database: Fulfillment status.

Use distinct shapes for databases and the queue where supported.
Keep labels readable and minimize crossing connectors.
Do not add components or relationships.
```

### Payment workflow with bounded retries

```text
Create a top-to-bottom flowchart with decision diamonds and labelled branches.

Start -> Charge payment -> Payment result?
- Approved -> Confirm order -> Accepted.
- Declined -> Notify payment failure -> Rejected.
- Retryable error -> Attempts remaining?

Allow at most three total payment attempts, including the initial attempt.
- Attempts remaining? Yes -> Wait 5 seconds -> Charge payment.
- Attempts remaining? No -> Notify payment failure -> Rejected.

Reuse the Charge payment, Notify payment failure, and Rejected nodes.
Keep the retry loop distinct from the success path. Do not add steps.
```

## Adding a new diagram skill

Select [add-diagrams-skill](.github/agents/add-diagrams-skill.agent.md) and provide
the tool or approach plus an upstream URL or local skill path:

```text
Add a new diagram skill from <source URL or local skill path>.
Use <display name> as its name and <backend-id> as its selection/output ID.
```

The agent adds the skill and updates its registration, prerequisites, and
provenance. New skills must provide editable/native output and a PNG/JPG preview
that can be included in the evaluation summary.

See the [workflow contract](.github/skills/COMPARISON.md) for integration
requirements and [sources and adaptations](.github/skills/SOURCES.md) for
provenance and licensing.
