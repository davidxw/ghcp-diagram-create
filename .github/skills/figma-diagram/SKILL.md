---
name: figma-diagram
description: "Use when create-diagrams requests the Figma MCP comparison output. Generate one FigJam diagram and persist its Mermaid source, returned link, and tool response."
user-invocable: false
disable-model-invocation: true
---

# Figma diagram comparison

This is a repository adapter to Figma's official `generate_diagram` workflow,
not a copy of the upstream skill (no redistribution license was declared).
See [sources](../SOURCES.md). Read and obey
[the comparison contract](../COMPARISON.md).

## Prerequisites

The official remote Figma MCP server must be enabled and already authenticated.
The workspace configuration names it `figma`. The agent allows only its
`generate_diagram` tool, not design-context, screenshots, code generation,
canvas editing, or other servers' Figma tools.

## Procedure

1. Use `run_dir/figma/`. If `figma/generate_diagram` is unavailable or requires
   authentication, record `blocked`; do not configure, authenticate, browse,
   invoke a router, or substitute a different server.
2. Independently translate the exact prompt into the Mermaid dialect accepted
   by Figma: flowchart, sequence, state, ER, or Gantt. Preserve the requested
   diagram type. Unsupported types (including class, C4, mindmap, pie, timeline,
   and git graphs) or content requiring unsupported annotations/styling are
   `blocked`, not approximated.
3. Use simple camelCase identifiers and quoted labels for special characters.
   The renderer does not accept HTML labels, emojis, or literal `\n` label
   escapes. Sequence notes and Gantt styling are not preserved. If the prompt
   requires those features, record `blocked` rather than silently dropping them.
   Do not use the optional architecture-layout mode: it imposes additional
   graph restrictions. Ordinary architecture flowcharts use the normal layout.
4. Write the independently authored source as `diagram.mmd`. Call
   `figma/generate_diagram` exactly once with the parameters exposed by its
   actual schema. The documented required fields are `name` and
   `mermaidSyntax`; use `name: "Diagram comparison"` and the saved source.
   Do not create a file separately, pass a made-up file key, invoke
   `use_figma`, or implement the upstream hybrid/enrichment workflow.
5. Save the returned text exactly as `response.txt`. Extract only the returned
   diagram URL and write `diagram.url` as a Windows Internet Shortcut:

   ```ini
   [InternetShortcut]
   URL=<the actual returned https://www.figma.com/board/... link>
   ```

   Accept a returned HTTPS link on `figma.com` or `www.figma.com`, including
   other diagram URL paths. Never fabricate a link or open it. A tool error or
   a response with no diagram link is `failed`, not successful local output.
6. Require all three files to exist and be nonempty before recording `created`
   in `status.json`. Keep the tool's diagnostic response on failure, but list
   no successful artifacts. Return only status and artifact paths.

The diagram lives in FigJam, not on disk. The local artifacts are its source,
link, and response. No local image export is claimed or attempted.
