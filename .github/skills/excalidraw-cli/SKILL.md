---
name: excalidraw-cli
description: "Create diagrams with Ahmad Awais's excalidraw-cli using element JSON, label shorthand, and checkpoints. Use for the Excalidraw-cli comparison approach, distinct from direct Excalidraw JSON authoring."
user-invocable: false
disable-model-invocation: true
---

# Excalidraw-cli

This is an independently written repository adapter to
[ahmadawais/excalidraw-cli](https://github.com/ahmadawais/excalidraw-cli/tree/d62ab7af3d7e7985724e80fa64a2253771a2513a),
not a vendored copy of its skill. See [source attribution](../SOURCES.md).
Follow [the shared scope and bookkeeping contract](../COMPARISON.md).

## Prerequisites and reference

- Require Node.js 18+ and an already installed `excalidraw-cli` executable.
  The package also exposes `excalidraw`; use that alias only after verifying its
  help/version identifies this CLI. Do not install the package or use
  dependency-downloading `npx` during a diagram run.
- Check the installed version and `create --help` for supported options. Read
  `excalidraw-cli --no-banner reference --raw` for its element syntax as needed.
  Save reference output and command logs inside this backend's directory.
- Consult the pinned [upstream skill](https://github.com/ahmadawais/excalidraw-cli/blob/d62ab7af3d7e7985724e80fa64a2253771a2513a/skills/skill.md)
  and [implementation](https://github.com/ahmadawais/excalidraw-cli/blob/d62ab7af3d7e7985724e80fa64a2253771a2513a/src/lib/elements.ts)
  when the installed reference leaves a concrete question unresolved.
- A compatible Excalidraw viewer/exporter or browser workflow is required to save
  the PNG/JPG preview as well as perform visual inspection and requested image
  exports. The CLI itself neither renders PNG/SVG nor calculates automatic
  graph layout. Its `export` command uploads a diagram to a sharing service.

## Authoring workflow

1. Work in `run_dir/excalidraw-cli/`. Independently author `elements.json` as a
   JSON array from the shared input. Do not reuse the direct Excalidraw backend's
   source or output. If the CLI is unavailable, record `blocked`; do not replace
   this approach with direct JSON authoring.
2. Choose coordinates, dimensions, text sizes, colors, and arrow paths for the
   requested diagram. Shape/arrow `label` shorthand is supported and expands into
   bound text; its optional fields include `text`, `fontSize`, `fontFamily`, and
   `strokeColor`. Supply real unique IDs and resolving arrow bindings.
3. Default to Standard `diagram.excalidraw`, with `roughness: 0` (Architect,
   the leftmost Sloppiness option), unless different sloppiness is explicitly
   requested. Set that value on authored elements rather than accepting the
   CLI's shape/arrow default of `2`. Do not change text or relationships to
   accommodate a convenience API.
4. Generate the native file with the actual CLI. For example, from this backend
   directory, substituting absolute paths:

   ```powershell
   excalidraw-cli --no-banner create "<absolute elements.json>" --output "<absolute diagram.excalidraw>" --checkpoint-dir "<absolute checkpoints directory>"
   ```

   Capture exit status and diagnostics. Keep checkpoints in this backend's
   `checkpoints` subdirectory. Any `checkpoint` command must also specify
   `--checkpoint-dir`; do not read/write the default home-directory store.
   Checkpoint restore/delete operations may assist refinement of this run's
   diagram. Keep the final authoring input so the result is reproducible.
5. After each generation, finalize the saved native JSON with a local JSON
   editor/script: apply the selected `roughness` to **every** final element,
   including generated label elements, and set
   `appState.currentItemRoughness` to the same value. The reviewed CLI rebuilds
   `appState` and `files`, so reapply any requested editor state or embedded media
   from the authored input as needed; do not silently discard these.
   Preserve the CLI's actual `source` (normally `"excalidraw-cli"`), not the
   direct-authoring skill's source convention.
6. Parse the saved file. Require `type: "excalidraw"`, `version: 2`, a nonempty
   `source`, an `elements` array, `appState`, and `files`. Check unique IDs,
   finite coordinates/dimensions/arrow points, resolving container/arrow/bound
   element references, and the selected element/editor roughness values. Verify
   labels, relationships, direction, and requested content. CLI JSON parsing
   alone is not full diagram validation.
7. Save a PNG/JPG view of the final CLI-generated native file, normally
   `diagram.png`, using an available compatible viewer/editor or browser workflow.
   Decode and inspect the image for clipped or overlapping text and incorrect
   connectors, and refine concrete defects.
   Regenerate through the CLI when changing its authoring input, then repeat
   finalization, validation, and image capture. Missing preview capability is
   `blocked`. Disclose unavailable visual inspection; do not claim the CLI itself
   rendered an image.
8. Honor explicit alternate output requests using a compatible exporter or
   wrapper while retaining the CLI-generated Standard source. If an export
   cannot be produced with available tools, report `blocked`, not native JSON
   presented as a completed image/animation export.
9. Record `elements.json`, `diagram.excalidraw`, the required PNG/JPG preview,
   any successful requested exports, and `status.json`. Use `tool: "excalidraw-cli"`
   and the shared status shape, including `preview` and `tokens_used`.
   Retain the preview for the HTML run summary.
   Checkpoints, logs, and technical references are supporting files, not final
   diagram artifacts.

## Optional cloud sharing

Only when the user explicitly requests or authorizes sharing/uploading:

```powershell
excalidraw-cli --no-banner export "<absolute diagram.excalidraw>"
```

This uploads an encrypted diagram to the Excalidraw service and returns a URL;
it is **not** a PNG/SVG export. Save the actual response as `response.txt` and
the returned HTTPS Excalidraw sharing link as `diagram.url`:

```ini
[InternetShortcut]
URL=<actual returned link>
```

Validate the returned link and files before listing them as artifacts. Selecting
Excalidraw-cli or requesting a local image export does not authorize an upload.
