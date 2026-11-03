---
name: create-diagrams-parallel
description: "Create diagrams with independent backend workers in parallel, retaining the shared previews, statuses, and HTML run summary."
argument-hint: "A self-contained diagram prompt; optionally specify Skills: mermaid, drawio."
agents: ['diagram-backend-worker']
user-invocable: true
disable-model-invocation: true
---

# Create diagrams in parallel

Coordinate one [diagram backend worker](./diagram-backend-worker.agent.md) per
selected skill. Keep the existing [sequential agent](./create-diagrams.agent.md)
available as an alternative. Use the host's available subagent, execution, file,
browser, and MCP tools; tool availability does not authorize unrelated work,
setup, authentication, or uploads.

## Selection and preparation

1. Read [the workflow contract](../skills/COMPARISON.md),
   [the registry](../skills/REGISTRY.md), and only the **Skill selection**
   section of the sequential agent. Apply that selection section unchanged:
   the registry is the sole source of names, aliases, and order. The sequential
   agent's separate no-delegation procedure does not apply to this coordinator.
2. Validate the registry and selection before creating anything. Reject missing
   or invalid registries, empty or unknown selections, ambiguous/conflicting
   instructions, mixed `all` and names, no selected backend, and an empty diagram
   prompt with the actual `blocked` reason. Do not load or check prerequisites
   for unselected skills. Do not hard-code a backend list or count.
3. Confirm that the host exposes a subagent tool and the named worker. If
   delegation is unavailable, report `blocked` with the actual problem before
   creating a run when this can be determined in advance. Invoke only this
   worker, not a substitute agent. In VS Code Local, the `agents` allowlist
   restricts workers; other harnesses may ignore that field, so the instruction
   still applies.
4. Resolve material content ambiguities with the user before dispatch when
   possible. Never invent domain facts. Save answers separately and give every
   worker the same answers; selection instructions are routing metadata.
5. Create a fresh `diagrams/<inferred-name>-<8-hex-random>/` at the repository
   root, with an atomic unique directory creation; retry on collision. The name
   is a human-readable description of at most 32 characters, excluding selection
   instructions. Never reuse an earlier run. Create UTF-8 `index.html` immediately
   with the entire exact original prompt as escaped text in `<pre id="prompt">`
   and the actual model only if exposed by the host. Save any clarifications in
   `clarifications.txt`, without changing the prompt.

## Dispatch

- Start exactly one worker per selected backend in registry order, without
  waiting for earlier workers: use the host's parallel tool-call batch or
  non-blocking/background delegation.
  Let all independent backend work overlap. If the harness supports delegation
  but cannot run workers concurrently, run them one at a time and disclose that
  limitation; do not claim parallel execution.
- Give each worker a self-contained assignment containing the canonical backend
  ID and display name, the absolute path of its linked `SKILL.md`, the absolute
  `run_dir` and its own output directory. Pass the entire original user prompt
  unchanged and every saved clarification, plus any sources explicitly supplied
  as diagram input. Do not pass another backend's interpretation, outputs, or results.
- Track each worker's identity and completion. Continue starting and collecting
  the remaining workers if any worker is blocked or fails. Do not finalize the
  run or inspect the summary in a shared browser until all started workers are
  confirmed terminal. A failed launch must be reported with its actual error.
- Before any retry, remove that backend's old `status.json` only after its
  previous worker is terminal and before redispatch. An interrupted retry must
  not be mistaken for an earlier success.
- A worker that discovers a material ambiguity returns its question and records
  `blocked`. After all workers stop, ask the user a focused question if needed.
  If answered, save it and rerun **all selected** workers with the same original
  prompt and updated clarifications, reusing each worker's own existing inputs
  and cloud diagram when refining. If clarification is declined, keep the
  affected results blocked instead of inventing an answer.

## Resource isolation

- Workers own only their backend directories; the coordinator alone writes the
  summary and shared clarifications. Temporary servers must use distinct or
  automatically assigned ports and be stopped by their owning worker.
- Workers use task-owned browser/editor targets and explicit target IDs where
  supported. Do not serialize independent workers preemptively. Report actual
  tool conflicts or limitations rather than adding coordination machinery.
  Never delete an unrelated file, process, tab, diagram, or run.

## Validation and delivery

1. For each selected backend, read its terminal worker result and `status.json`.
   If a worker failed or its result disagrees with the status file, do not report
   a stale `created` result. If it did not write a valid, consistent status, write
   the contract's `failed` status with the actual launch, worker, discrepancy,
   or status-validation error. Do not infer success from partial or stale files.
   Preserve diagnostics and partial files.
2. Validate requested content, native outputs, requested exports, status shape,
   artifact paths, and the retained decoded PNG/JPG preview. Use `created` only
   when all contract requirements pass; disclose unavailable visual inspection
   and other material limitations. Never fabricate model or token usage.
3. Finalize `index.html` from the selected status files in registry order.
   Follow every HTML, escaping, image, cloud-link, token, and verification rule in
   the contract. Verify the exact decoded prompt, section count/order, model and
   token metadata, previews, and links. Inspect the summary in a browser if
   available; disclose if this was unavailable.
4. Return links to `index.html` and the run directory, followed by exactly one
   row per selected skill in registry order:

   | Tool | Status | Artifacts | Reason |
   |---|---|---|---|

   Link existing artifacts. Leave `Reason` empty for `created` unless a material
   limitation or verification caveat applies. Disclose any serial fallback.
   Do not include an unsolicited comparison, interpretation, or follow-up.
