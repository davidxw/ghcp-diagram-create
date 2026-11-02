---
name: excalidraw-diagram
user-invocable: false
disable-model-invocation: true
description: Generate Excalidraw diagrams from text content. Supports three output modes - Standard (.excalidraw, default), Obsidian (.md), and Animated (.excalidraw with animation order). Triggers on "Excalidraw", "draw", "flowchart", "mind map", "visualize", "diagram", "standard excalidraw", "obsidian excalidraw", "animated excalidraw", "animated diagram", "animate".
metadata:
  version: 1.2.1
---

# Excalidraw Diagram Generator

## Repository workflow

Adapted from Axton Liu's excalidraw-diagram skill (MIT); see [source attribution](../SOURCES.md).
Follow the normal technical workflow below within
[the shared scope and bookkeeping contract](../COMPARISON.md).

1. Work in `run_dir/excalidraw/`. Honor an explicit output mode/format. Otherwise
   use **Standard** mode by default; Obsidian and Animated require an explicit
   request. Tool-selection metadata alone does not request a mode. Use `diagram.md` for Obsidian,
   `diagram.excalidraw` for Standard, or `diagram.animate.excalidraw` for Animated.
2. Independently author the native output from the shared input using the element
   template, design rules, and [schema reference](./references/excalidraw-schema.md).
   Preserve labels exactly; escape quotes/parentheses using JSON syntax,
   not the upstream substitutions. An available compatible editor/rendering tool
   may assist authoring. A compatible viewer/exporter or browser workflow is
   required to save the PNG/JPG preview, even for native JSON/Markdown output.
3. Default every element to `roughness: 0`: the leftmost **Architect**
   selection under **Sloppiness**, with clean strokes for comparison with
   other tools. Also set `appState.currentItemRoughness: 0` so newly drawn
   elements use the same setting. Only use a different sloppiness value when
   explicitly requested in the prompt; keep both settings consistent.
4. Parse the saved JSON locally, extracting it from the Markdown wrapper in
   Obsidian mode. Require `type: "excalidraw"`, `version: 2`, an `elements`
   array, `appState`, and `files`, with the mode's `source` below. Verify element
   IDs are unique, coordinates/dimensions are finite,
   and arrow/container references resolve. Verify element `roughness` and
   `appState.currentItemRoughness` match the default or explicitly requested
   sloppiness. In Animated mode, also check animation order/duration metadata.
   Correct invalid data and revalidate rather than stopping at the first defect.
5. Save a PNG/JPG view of the final native diagram, normally `diagram.png`,
   with an available compatible editor, renderer, or browser workflow.
   Decode the image and inspect text, bindings, arrows, spacing, and bounds;
   refine and regenerate as needed. Missing preview capability is `blocked`. Obsidian
   viewing requires its Excalidraw plugin; native Standard authoring does not
   require an installed application. Follow the shared permission policy before
   uploading to a hosted editor. Requested image/animation exports require a
   working compatible exporter, not just animation metadata or a link.
6. Record the chosen native file, required PNG/JPG preview, requested successful
   exports, and `status.json`, including `preview` and `tokens_used` per the shared
   contract. Retain the preview for the HTML run summary.
   Disclose unavailable visual inspection. Use the backend directory instead of
   upstream current-directory defaults, and omit unsolicited explanatory output.

## Upstream technical guidance

Create Excalidraw diagrams from text content with multiple output formats.

## Output Modes

Honor an explicitly requested output mode; otherwise use Standard. Generic
diagram words and tool-selection metadata do not request Obsidian or Animated.

| Trigger words | Output mode | File format | Usage |
|--------|----------|----------|------|
| No explicit mode, or `standard excalidraw` | **Standard** (default) | `.excalidraw` | Open, edit, or share on excalidraw.com |
| Explicit Obsidian output request | **Obsidian** | `.md` | Open directly in Obsidian |
| Explicit animation request, such as `animated excalidraw` or `animate` | **Animated** | `.excalidraw` | Drag into excalidraw-animate to generate an animation |

## Workflow

1. **Honor the requested output mode**, or default to Standard (see Output Modes above)
2. Analyze content - identify concepts, relationships, hierarchy
3. Choose diagram type (see Diagram Types below)
4. Generate Excalidraw JSON (add animation order if Animated mode)
5. Output in correct format based on mode
6. **Automatically save to current working directory**
7. Notify user with file path and usage instructions

## Output Formats

### Mode 1: Obsidian Format (Explicit Request Only)

**Output exactly the following structure without modification:**

````markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'

# Excalidraw Data

## Text Elements
%%
## Drawing
```json
{Complete JSON data}
```
%%
````

**Key points:**
- Frontmatter must include `tags: [excalidraw]`
- Preserve the complete warning message
- Enclose the JSON in `%%` markers
- Do not use any other value for the `excalidraw-plugin` frontmatter setting besides `parsed`
- **File extension**: `.md`

### Mode 2: Standard Excalidraw Format (Default)

Output a plain JSON file that can be opened on excalidraw.com:

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [...],
  "appState": {
    "gridSize": null,
    "currentItemRoughness": 0,
    "viewBackgroundColor": "#ffffff"
  },
  "files": {}
}
```

**Key points:**
- Use `https://excalidraw.com` for `source` (not the Obsidian plugin)
- Use plain JSON without a Markdown wrapper
- **File extension**: `.excalidraw`

### Mode 3: Animated Excalidraw Format

Use the Standard format, but add `customData.animate` to each element to control animation order:

```json
{
  "id": "element-1",
  "type": "rectangle",
  "customData": {
    "animate": {
      "order": 1,
      "duration": 500
    }
  },
  ...other standard fields
}
```

**Animation order rules:**
- `order`: Animation sequence (1, 2, 3...); lower numbers appear first
- `duration`: Drawing duration for the element in milliseconds; defaults to 500
- Elements with the same `order` appear simultaneously
- Recommended sequence: title -> main structure -> connectors -> detail text

**Usage:**
1. Generate the `.excalidraw` file
2. Drag it into https://dai-shi.github.io/excalidraw-animate/
3. Click Animate to preview, then export SVG or WebM

**File extension**: `.excalidraw`

---

## Diagram Types & Selection Guide

Choose an appropriate diagram type to improve understanding and visual appeal.

| Diagram type | Use cases | Approach |
|--------------|-----------|----------|
| **Flowchart** | Step-by-step instructions, workflows, task execution order | Connect steps with arrows to clearly show the flow |
| **Mind Map** | Exploring concepts, categorizing topics, capturing ideas | Use a radial structure branching outward from a central topic |
| **Hierarchy** | Organizational structures, content levels, system decomposition | Arrange hierarchical nodes top-to-bottom or left-to-right |
| **Relationship** | Influences, dependencies, and interactions between elements | Connect shapes with lines, arrows, and explanations |
| **Comparison** | Comparing two or more options or viewpoints | Use side-by-side columns or a table with explicit comparison dimensions |
| **Timeline** | Event progression, project progress, model evolution | Arrange key dates and events along a time axis |
| **Matrix** | Two-dimensional categorization, task prioritization, positioning | Place items on a coordinate plane with X and Y dimensions |
| **Freeform** | Unstructured content, idea capture, initial information gathering | Place shapes and arrows freely without structural constraints |

## Design Rules

### Text & Format
- **All text elements must use** `fontFamily: 5` (the Excalifont handwriting font)
- **Label fidelity**: Preserve quotes, parentheses, and other supplied characters;
  use JSON escaping rather than substituting punctuation
- **Font sizes** (hard minimums; smaller text is unreadable at normal zoom):
  - Titles: 20-28px (minimum 20px)
  - Subtitles: 18-20px
  - Body text/labels: 16-18px (minimum 16px)
  - Secondary annotations: 14px (only for nonessential supporting details; use sparingly)
  - **Never use sizes below 14px**
- **Line height**: Use `lineHeight: 1.25` for all text
- **Estimating centered text**: Standalone text elements do not center automatically; calculate the x coordinate manually:
  - Estimated width: `estimatedWidth = text.length * fontSize * 0.5` (use `* 1.0` for CJK characters)
  - Centering formula: `x = centerX - estimatedWidth / 2`
  - Example: To center "Hello" (5 characters, fontSize 20) at x=300, use `estimatedWidth = 5 * 20 * 0.5 = 50`, then `x = 300 - 25 = 275`

### Layout & Design
- **Canvas bounds**: Keep all elements within a 0-1200 x 0-800 area
- **Minimum shape size**: Rectangles and ellipses containing text must be at least 120x60px
- **Element spacing**: Leave at least 20-30px between elements to prevent overlap
- **Clear hierarchy**: Use different colors and shapes to distinguish information levels
- **Graphical elements**: Organize information with rectangles, circles, arrows, and other suitable shapes
- **No emoji**: Do not use emoji in diagram text; use simple shapes (circles, squares, arrows) or colors for visual markers

### Color Palette

**Text colors (`strokeColor` for text):**

| Purpose | Color | Description |
|------|------|------|
| Titles | `#1e40af` | Deep blue |
| Subtitles/connectors | `#3b82f6` | Bright blue |
| Body text | `#374151` | Dark gray (on white, use no lighter than `#757575`) |
| Emphasis | `#f59e0b` | Gold |

**Shape fill colors (`backgroundColor`, `fillStyle: "solid"`):**

| Color | Meaning | Use cases |
|------|------|---------|
| `#a5d8ff` | Light blue | Inputs, data sources, main nodes |
| `#b2f2bb` | Light green | Success, outputs, completed items |
| `#ffd8a8` | Light orange | Warnings, pending items, external dependencies |
| `#d0bfff` | Light purple | Processing, middleware, special items |
| `#ffc9c9` | Light red | Errors, critical items, alerts |
| `#fff3bf` | Light yellow | Notes, decisions, planning |
| `#c3fae8` | Light teal | Storage, data, caches |
| `#eebefa` | Light pink | Analysis, metrics, statistics |

**Region background colors (large rectangles with `opacity: 30` for layered diagrams):**

| Color | Meaning |
|------|------|
| `#dbe4ff` | Frontend/UI layer |
| `#e5dbff` | Logic/processing layer |
| `#d3f9d8` | Data/tools layer |

**Contrast rules:**
- On white backgrounds, use text no lighter than `#757575` to keep it readable
- Use dark text variants on light fills (for example, `#15803d` rather than `#22c55e` on light green)
- Avoid light gray text (`#b0b0b0`, `#999`) on white backgrounds

Reference: [references/excalidraw-schema.md](references/excalidraw-schema.md)

## JSON Structure

**Obsidian mode:**
```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://github.com/zsviczian/obsidian-excalidraw-plugin",
  "elements": [...],
  "appState": { "gridSize": null, "currentItemRoughness": 0, "viewBackgroundColor": "#ffffff" },
  "files": {}
}
```

**Standard / Animated modes:**
```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [...],
  "appState": { "gridSize": null, "currentItemRoughness": 0, "viewBackgroundColor": "#ffffff" },
  "files": {}
}
```

## Element Template

Use the fields below for authored elements. Preserve valid native metadata when
using an editor/exporter. `boundElements` is `null` when there are no bindings;
otherwise use the actual text/arrow bindings described in the schema reference.

```json
{
  "id": "unique-id",
  "type": "rectangle",
  "x": 100, "y": 100,
  "width": 200, "height": 50,
  "angle": 0,
  "strokeColor": "#1e1e1e",
  "backgroundColor": "transparent",
  "fillStyle": "solid",
  "strokeWidth": 2,
  "strokeStyle": "solid",
  "roughness": 0,
  "opacity": 100,
  "groupIds": [],
  "roundness": {"type": 3},
  "seed": 123456789,
  "version": 1,
  "isDeleted": false,
  "boundElements": null,
  "updated": 1,
  "link": null,
  "locked": false
}
```

`strokeStyle` values: `"solid"` (default), `"dashed"`, or `"dotted"`. Dashed lines suit optional paths, asynchronous flows, and weak relationships.

Text elements add:
```json
{
  "text": "Display text",
  "fontSize": 20,
  "fontFamily": 5,
  "textAlign": "center",
  "verticalAlign": "middle",
  "containerId": null,
  "originalText": "Display text",
  "autoResize": true,
  "lineHeight": 1.25
}
```

**Animated mode additionally uses** a `customData` field:
```json
{
  "id": "title-1",
  "type": "text",
  "customData": {
    "animate": {
      "order": 1,
      "duration": 500
    }
  },
  ...other fields
}
```

See [references/excalidraw-schema.md](references/excalidraw-schema.md) for all element types.

---

## Additional Technical Requirements

### Text Elements Handling
- The `## Text Elements` section in Markdown **must remain empty**, with only `%%` as the delimiter
- The Obsidian Excalidraw plugin **automatically populates text elements** from the JSON data
- Do not manually list all text content

### Coordinates & Layout
- **Coordinate system**: The top-left corner is the origin (0,0)
- **Recommended bounds**: Keep all elements within 0-1200 x 0-800 pixels
- **Element IDs**: Each element needs a unique `id` (a string such as `title` or `box1`)

### Required Fields for All Elements

Use `boundElements: null` for unbound elements, or a list of resolving bindings
when text/arrows are attached. Editor-generated native metadata may be retained.

```json
{
  "id": "unique-identifier",
  "type": "rectangle|text|arrow|ellipse|diamond",
  "x": 100, "y": 100,
  "width": 200, "height": 50,
  "angle": 0,
  "strokeColor": "#color-hex",
  "backgroundColor": "transparent|#color-hex",
  "fillStyle": "solid",
  "strokeWidth": 2,
  "strokeStyle": "solid|dashed|dotted",
  "roughness": 0,
  "opacity": 100,
  "groupIds": [],
  "roundness": {"type": 3},
  "seed": 123456789,
  "version": 1,
  "isDeleted": false,
  "boundElements": null,
  "updated": 1,
  "link": null,
  "locked": false
}
```

### Text-Specific Properties
Text elements (`type: "text"`) require these additional properties:
```json
{
  "text": "Display text",
  "fontSize": 20,
  "fontFamily": 5,
  "textAlign": "center",
  "verticalAlign": "middle",
  "containerId": null,
  "originalText": "Display text",
  "autoResize": true,
  "lineHeight": 1.25
}
```

### appState Configuration
```json
"appState": {
  "gridSize": null,
  "currentItemRoughness": 0,
  "viewBackgroundColor": "#ffffff"
}
```

### files Field
```json
"files": {}
```

## Common Mistakes to Avoid

- **Misaligned text** - A standalone text element's `x` is its left edge, not its center. Calculate it with the centering formula to avoid shifting text to one side
- **Overlapping elements** - Elements with similar y coordinates can overlap. Before placing a new element, check for at least 20px of clearance from surrounding elements
- **Insufficient canvas padding** - Do not place content against the canvas edges; leave 50-80px of padding on all sides
- **Titles not centered over the diagram** - Center titles over the full width of the diagram below, rather than fixing them at x=0
- **Overflowing arrow labels** - Long labels (such as "ATP + NADPH") can exceed short arrows. Keep labels short or increase arrow length
- **Insufficient contrast** - Light text is almost invisible on white. Use text no lighter than `#757575`, and use dark variants for colored text
- **Text too small** - Sizes below 14px are unreadable at normal zoom; body text must be at least 16px

## Implementation Notes

### Auto-save & File Generation Workflow

When generating an Excalidraw diagram, **automatically perform the following steps**:

#### 1. Choose an Appropriate Diagram Type
- Refer to the Diagram Types & Selection Guide above based on the supplied content
- Identify the content's main purpose and choose the most suitable visualization

#### 2. Generate a Meaningful Filename

Choose the file extension based on the output mode:

| Mode | Filename format | Example |
|------|-----------|------|
| Obsidian | `[topic].[type].md` | `business-model.relationship.md` |
| Standard | `[topic].[type].excalidraw` | `business-model.relationship.excalidraw` |
| Animated | `[topic].[type].animate.excalidraw` | `business-model.relationship.animate.excalidraw` |

- Prefer clear, descriptive English filenames

#### 3. Automatically Save the File with the Write Tool
- **Save location**: The current working directory (detected from environment variables)
- **Full path**: `{current_directory}/[filename].md`
- This keeps paths portable without hardcoding them

#### 4. Ensure the Markdown Structure Is Correct
**Generate exactly the following format without modification:**

````markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'

# Excalidraw Data

## Text Elements
%%
## Drawing
```json
{Complete JSON data}
```
%%
````

#### 5. JSON Data Requirements
- Include the complete Excalidraw JSON structure
- Use `fontFamily: 5` for all text elements
- Preserve supplied punctuation and escape it correctly in JSON
- JSON must be valid and pass syntax validation
- Every element must have a unique `id`
- Include `appState` and `files: {}` fields

#### 6. User Feedback & Confirmation
Tell the user:
- The diagram has been generated
- Its exact save location
- How to view it in Obsidian
- The diagram's design choices (which diagram type was selected and why)
- Whether adjustments or modifications are needed

### Example Output Messages

**Obsidian mode:**
```
Excalidraw diagram generated!

Saved to: business-model.relationship.md

Usage:
1. Open this file in Obsidian
2. Click the MORE OPTIONS menu in the top-right corner
3. Select Switch to EXCALIDRAW VIEW
```

**Standard mode:**
```
Excalidraw diagram generated!

Saved to: business-model.relationship.excalidraw

Usage:
1. Open https://excalidraw.com
2. Click the top-left menu -> Open -> select this file
3. Alternatively, drag the file directly onto the excalidraw.com page
```

**Animated mode:**
```
Animated Excalidraw diagram generated!

Saved to: business-model.relationship.animate.excalidraw

Animation order: title (1) -> main structure (2-4) -> connectors (5-7) -> explanatory text (8-10)

Generate the animation:
1. Open https://dai-shi.github.io/excalidraw-animate/
2. Click Load File and select this file
3. Preview the animation
4. Click Export to export SVG or WebM
```
