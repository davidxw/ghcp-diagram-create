---
name: create-diagrams-parallel
description: "Prepare one consistent diagram description, then run independent backend workers in parallel, retaining previews, statuses, and an HTML summary with original and prepared prompts."
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
4. Perform the contract's
   [shared prompt pre-evaluation](../skills/COMPARISON.md#shared-prompt-pre-evaluation)
   once in the coordinator, before dispatch. Keep `original_prompt` unchanged;
   use it verbatim as `diagram_prompt` if sufficiently concrete, otherwise
   generate one grounded, backend-neutral description. Resolve material
   ambiguities with the user, save answers separately, and incorporate them
   consistently. A grounded expansion proceeds without separate approval.
   Block unresolved preparation before creating a run. Never delegate this step
   to a worker; selection instructions remain routing metadata.
5. Create a fresh `diagrams/<inferred-name>-<8-hex-random>/` at the repository
   root, with an atomic unique directory creation; retry on collision. The name
   is a human-readable description of at most 32 characters, excluding selection
   instructions. Never reuse an earlier run. Create UTF-8 `index.html` immediately
   with the entire exact `original_prompt` as escaped text in `<pre id="prompt">`
   and the exact `diagram_prompt` in `<pre id="diagram-prompt">`. Always include
   both blocks, even when identical, plus the preparation outcome/explanation
   and the actual model only if exposed by the host. Save any clarifications
   in `clarifications.txt`, without changing the original input.

## Dispatch

- Start exactly one worker per selected backend in registry order, without
  waiting for earlier workers: use the host's parallel tool-call batch or
  non-blocking/background delegation.
  Let all independent backend work overlap. If the harness supports delegation
  but cannot run workers concurrently, run them one at a time and disclose that
  limitation; do not claim parallel execution.
- Give each worker a self-contained assignment containing the canonical backend
  ID and display name, the absolute path of its linked `SKILL.md`, the absolute
  `run_dir` and its own output directory. Pass the exact same shared
  `diagram_prompt` unchanged and every saved clarification, plus any sources
  explicitly supplied as diagram input. The original input remains provenance
  in the summary, not an alternative drawing specification. Do not pass another
  backend's interpretation, outputs, or results, or let a worker independently
  expand the subject content.
- Track each worker's identity and completion. Continue starting and collecting
  the remaining workers if any worker is blocked or fails. Do not finalize the
  run or inspect the summary in a shared browser until all started workers are
  confirmed terminal. A failed launch must be reported with its actual error.
- Before any retry, remove that backend's old `status.json` only after its
  previous worker is terminal and before redispatch. An interrupted retry must
  not be mistaken for an earlier success.
- A worker that discovers a material ambiguity returns its question and records
  `blocked`. After all workers stop, ask the user a focused question if needed.
  If answered, save it, revise and recheck the shared description once, and
  update the summary's prepared prompt and preparation outcome/explanation,
  leaving the original input unchanged. Invalidate every selected backend's
  old status before rerunning **all selected** workers with the exact updated
  `diagram_prompt` and clarifications, reusing each worker's own existing
  inputs and cloud diagram when refining. Never finalize results from an earlier
  description as current. If clarification is declined, leave the description
  unchanged and keep the affected results blocked instead of inventing an answer.

## Resource isolation

- Workers own only their backend directories; the coordinator alone prepares
  the shared description and writes the summary and shared clarifications.
  Temporary servers must use distinct or
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
2. Validate content against the shared description and original requirements,
   native outputs, requested exports, status shape,
   artifact paths, and the retained decoded PNG/JPG preview. Use `created` only
   when all contract requirements pass; disclose unavailable visual inspection
   and other material limitations. Never fabricate model or token usage.
3. Finalize `index.html` from the selected status files in registry order.
   Follow every HTML, escaping, image, cloud-link, token, and verification rule in
   the contract. Verify both exact decoded prompt blocks, the description
   actually used by every worker, preparation outcome/explanation, section
   count/order, model and token metadata, previews, and links. Inspect the
   summary in a browser if available; disclose if this was unavailable.
4. Return links to `index.html` and the run directory, followed by exactly one
   row per selected skill in registry order:

   | Tool | Status | Artifacts | Reason |
   |---|---|---|---|

   Link existing artifacts. Leave `Reason` empty for `created` unless a material
   limitation or verification caveat applies. Disclose any serial fallback.
   Do not include an unsolicited comparison, interpretation, or follow-up.
