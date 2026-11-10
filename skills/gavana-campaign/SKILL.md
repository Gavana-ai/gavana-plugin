---
name: gavana-campaign
description: Plan and run chat-first Gavana product campaigns. Use when a user gives a product, broad campaign request, visual references, wants creative directions, or asks to generate a selected campaign.
---

# Gavana Campaign

Gavana owns the durable campaign state and execution. Chat is the primary
interface; Canvas is the detail workspace a person can open when needed.

## Stage 1: prepare, then stop

For a broad campaign request, prepare the work before starting any paid image,
video, Template, or Action run.

1. Call `work_prepare` with the request, supplied images, and existing Gavana
   Canvas. Keep exact product or brand references as durable `node:` or
   `asset:` handles.
2. Infer a concise brief: product truth, audience, goal, deliverables, and
   visible assumptions. Ask one short question only when a missing factual
   detail would materially change the work; propose creative preferences.
3. Return exactly three materially different directions. Each must change the
   core idea, camera or perspective, composition, lighting, motion or texture,
   and copy hierarchy. Reject three centered hero-product shots with only
   background or color swaps. For cross-brand work, preserve both brand worlds
   and durable product identity references. Recommend by the stated goal, not
   aesthetic sameness.
4. Save the brief, directions, and references to the Gavana Canvas work, then
   stop. Offer: use the recommendation, choose a direction, or adjust. Use
   `work_get` to read the latest durable state and `work_continue` for a
   choice, answer, or adjustment.

Do not generate exploratory images just to fill missing references. A request
to “create” or “generate a campaign” does not select a direction by itself.

## Stage 2: execute one chosen direction

Call `work_execute` only after the user chooses a direction in the current
conversation, or explicitly asks for autopilot, and explicitly confirms the
selected deliverables. Autopilot means Gavana may use its recommendation; it
never means silently run all three directions.

- Run only the selected branch and requested deliverables.
- `work_execute` requires the selected direction, explicit confirmation, and
  one stable idempotency key per paid attempt. Never automatically retry a
  terminal failure or ambiguous provider response.
- Return the immediate durable progress; do not wait or automatically retry
  paid work. Use read-only `work_get` for a snapshot. When the owner needs to
  poll and reconcile provider progress, use write-authorized `work_refresh`.
  Preserve
  successful outputs after partial failure and make an individual failed output
  retryable.
- Save outputs and progress back to the same Gavana work so Chat and Canvas
  remain synchronized.

## Canvas detail work

Open Canvas only for visual inspection or manual changes. Preserve references,
connections, and existing layout; do not delete or reorganize unrelated work.
When a Canvas edit changes an intermediate result, mark only affected
downstream outputs outdated—never regenerate them without new user intent.

Before claiming completion, read Gavana's completion review and ensure the
requested outputs are durable. Report the result locations, selected direction,
and any failed or pending output plainly.
