---
name: figma-diagram
description: "Create a FigJam diagram through the official Figma MCP workflow, including supported layout, inspection, refinement, and diagram-specific annotations."
user-invocable: false
disable-model-invocation: true
---

# Figma diagram workflow

This is a repository adapter to Figma's official `generate_diagram` workflow,
not a copy of the upstream skill (no redistribution license was declared).
See [sources](../SOURCES.md). Follow
[the shared scope and bookkeeping contract](../COMPARISON.md).

## Prerequisites

The official remote Figma MCP server must be enabled and already authenticated.
The repository MCP configuration names it `figma`; use the actual tools exposed
by the configured official server. `generate_diagram` is the normal base route.
Inspection, screenshots, and FigJam editing tools such as `get_figjam` or
`use_figma` may be used when available and relevant to completing this diagram.
Do not assume those optional tools are present or call unrelated code-generation
or design workflows. A supported image-export/screenshot route or browser view
of the actual returned board is required to save the PNG/JPG preview.

## Technical guidance

Consult the official [diagram guide](https://github.com/figma/mcp-server-guide/blob/f3f264d9cbae059d9b96289ee33ce444b5c8e7e6/skills/figma-generate-diagram/SKILL.md)
and the relevant type reference before authoring. Its
[architecture layout](https://github.com/figma/mcp-server-guide/blob/f3f264d9cbae059d9b96289ee33ce444b5c8e7e6/skills/figma-generate-diagram/references/architecture.md)
and [hybrid workflow](https://github.com/figma/mcp-server-guide/blob/f3f264d9cbae059d9b96289ee33ce444b5c8e7e6/skills/figma-generate-diagram/references/workflow.md)
are available when appropriate. These links are references, not vendored copies.

## Procedure

1. Work in `run_dir/figma/`. Discover the official server's available tools and
   actual schemas. If no supported authenticated route can create the requested
   diagram, record `blocked`; do not configure or authenticate services.
2. Independently translate the shared input into the supported diagram type.
   `generate_diagram` accepts flowchart, sequence, state, ER, and Gantt syntax;
   it is not a general Mermaid renderer. If a requested type is unsupported,
   use a documented native FigJam route only when it can faithfully represent
   that type and the necessary tools are available. Otherwise report `blocked`,
   not a different diagram type.
3. Use simple camelCase identifiers and quoted labels for special characters.
   The renderer does not accept HTML labels, emojis, or literal `\n` label
   escapes. Sequence notes and Gantt styling are not preserved by the base
   renderer; use the documented hybrid FigJam workflow for required annotations
   or colors rather than silently dropping them. If extension tools required
   to meet the request are unavailable, report `blocked`, not an incomplete
   diagram marked as successful.
4. Choose the normal or architecture layout according to the official reference
   and actual tool schema. Use architecture layout only when its graph and
   direction constraints fit the input; never invent gateway nodes or alter
   relationships to satisfy it. Normal flowchart layout remains an alternative.
5. Save the independently authored Mermaid as `diagram.mmd`, or save the native
   authoring script when using a documented direct FigJam route. Call the real
   tool using its schema. `generate_diagram` requires `name` and `mermaidSyntax`;
   use a descriptive name. Do not create a separate file before this tool or
   invent a `fileKey`.
6. Save a local PNG/JPG view of the actual returned diagram, normally `diagram.png`,
   using a supported image export, screenshot response, or browser screenshot.
   Save actual image bytes, not just a remote URL or tool attachment identifier.
   Decode and inspect the complete diagram. Correct specific source/layout
   defects and regenerate the preview if needed; use supported
   FigJam extensions for annotations and styling that serve the request.
   Reuse the actual returned `fileKey` for refinements in the same board. Only
   replace nodes created during this run; do not delete pre-existing user work
   or create a trail of unnecessary draft boards.
7. Preserve each tool response unchanged (use numbered files for intermediate
   responses and `response.txt` for the final one). Preserve extension scripts
   and their responses too. Extract the actual returned diagram URL and write
   `diagram.url` as a Windows Internet Shortcut:

   ```ini
   [InternetShortcut]
   URL=<the actual returned https://www.figma.com/board/... link>
   ```

   Accept a returned HTTPS link on `figma.com` or `www.figma.com`, including
   other diagram URL paths. Never fabricate a link. Opening/inspecting the
   selected cloud diagram is allowed. Resolve tool errors through the normal
   workflow; an unresolved error or missing diagram link is `failed`.
8. Require the actual authoring source, shortcut, final response, and validated
   PNG/JPG preview to exist and be nonempty before recording `created` in
   `status.json`, including `preview` and `tokens_used` per the shared contract.
   Retain the preview for the HTML run summary. Any requested extension/export
   must also be completed and verified. If no supported route can save the
   preview, record `blocked` even when the board was created. Preserve the
   returned board reference for diagnosis/reuse, without advertising success.
   Keep diagnostics on failure without listing partial files as successful artifacts.

The editable diagram lives in FigJam. The local artifacts retain its source,
link, responses, and a required local PNG/JPG preview. A cloud link alone does
not complete the workflow; never claim an unsaved image exists.
