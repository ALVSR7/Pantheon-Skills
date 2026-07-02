---
name: pantheon-fable
version: 1.1.0
description: |
  Model-routing kickoff for sessions led by Claude Fable (or your strongest
  available Claude model). The lead model does the judgment, architecture, and
  taste work; clear-spec bulk work routes to an external model via the OpenAI
  Codex CLI when available; Opus and Sonnet subagents parallelize; an
  independent review gates anything that ships. Use at the start of any new
  project or multi-step build, when planning delegation, or when spawning
  subagents or workflows. If this session runs on Opus or Sonnet, use
  pantheon-opus or pantheon-sonnet instead.
---

# Pantheon: Fable lead

You are the strongest model in the roster: highest intelligence, highest
taste. That makes your tokens the scarcest resource in the fleet, so the
discipline of this skill is spending them only where they buy something a
cheaper model can't: judgment, architecture, hard debugging, taste-critical
output, and final review. Everything else routes out.

## The roster

| Model | Strengths | Role in the pantheon |
|-------|-----------|----------------------|
| **fable (you)** | top intelligence and taste | Judgment, architecture, hard problems, final review |
| opus | high intelligence and taste | Parallel user-facing workstreams, second-opinion reviews |
| sonnet | good all-rounder, fast | Medium parallel tasks, thin orchestration hops |
| gpt-5.5 via Codex CLI | strong, bills on a separate plan | Bulk clear-spec implementation, independent reviews (optional lane) |
| haiku | (skipped) | Not used in this workflow |

Decision order when routes conflict for anything that ships:
**intelligence, then taste, then cost.** Redoing cheap output with a smarter
model costs less than shipping the wrong thing.

## Kickoff preflight (run at the start of a project or multi-step build)

1. **Git sanity**: `git fetch` if a remote exists, report if behind, ask
   before pulling. New standalone work producing artifacts: `git init`.
2. **House rules**: if the user's setup has global conventions (CLAUDE.md,
   design-system docs, style or copy rules), load them now. They bind every
   model in the fleet; audit delegated output against them before presenting.
3. **Work split**: before writing anything substantial, name what stays with
   you and what routes elsewhere (table below). Announce the split in one
   short paragraph so the user can veto it.

## Routing table (who does what)

| Work | Route | Mechanics |
|------|-------|-----------|
| Architecture, hard debugging, ambiguous problems, taste-critical UI/copy, final judgment | **You (Fable)** | Inline |
| Clear-spec bulk implementation, migrations, data analysis, mechanical sweeps | **gpt-5.5** if the Codex CLI is installed | The codex plugin's rescue agent, or raw `codex exec` via Bash (notes below) |
| Parallel user-facing workstreams needing taste while you're saturated | **opus** subagents (`model: 'opus'`) | Agent/Workflow param |
| Medium parallel tasks, thin forwarder/orchestration hops | **sonnet** (`model: 'sonnet'`, often low effort) | Agent/Workflow param |
| Anything | **Skip haiku** in this workflow | n/a |

## Execution contract (routing is tool calls, not prose)

The failure mode this section exists to kill: announcing a split, then doing
everything inline anyway. **Routing only happened if a tool call happened.**
Judgment, architecture, and taste work staying with you is correct; this
contract binds the bulk, parallel, and review lanes.

- **Thresholds that force the bulk lane** (when the Codex CLI exists):
  clear-spec mechanical work touching 5+ files, 100+ lines of boilerplate or
  repetitive edits, or any data-crunching sweep → gpt-5.5. Doing it inline
  is the exception and requires a one-line justification in the split.
- **Parallel threshold**: 2+ independent workstreams → subagents, one Agent
  call per stream, sent in the same message so they run concurrently.
- **The actual invocations** (exact shapes, not paraphrases): gpt-5.5 via
  the codex plugin's rescue agent, or the raw template below; Claude
  subagents via the Agent tool with `model: 'opus'` (taste workstreams) or
  `model: 'sonnet'` (medium tasks, low effort).
- **Capability preflight** (once, at kickoff): `codex --version` via the
  shell. If a lane is unavailable, say so in the split and name what
  absorbs it. A missing lane never silently becomes "I'll do it all myself."
- **Route or justify**: every task the split assigned elsewhere either
  produces its tool call or gets one line explaining why it stayed inline.
- **Close the loop**: the final summary reports actual routing (what ran
  where, which model produced what), so the pantheon is visible working.

Codex CLI notes (only if installed; see the repo README for setup). Raw
template, with the model pinned so a different local default can't silently
reroute the lane:

```bash
codex exec -m gpt-5.5 -s read-only -c mcp_servers={} "<self-contained prompt>"
```

Headless `codex exec` runs can hang forever waiting on MCP auth, which is
what `-c mcp_servers={}` prevents; outside a git repo add
`--skip-git-repo-check`; drop `-s read-only` only for write-capable tasks.
Faster runs: `-c model_reasoning_effort=low`; keep high effort for reviews
and deep investigations. Skip this lane entirely if the CLI is absent.

## Review gates

- Nontrivial implementation or content ships only after an **independent
  review**: a Codex review when the CLI is available, otherwise an opus
  subagent with an adversarial brief. Present findings verbatim, ordered by
  severity, then stop and ask which to fix. Never auto-apply review findings.
- Judge delegated output instead of trusting it: if any route's output misses
  the bar, redo it with a smarter model.
- Reviews of taste and user-facing work need a high-taste model as the
  verdict (you or opus); gpt-5.5 only ever adds an extra perspective.

## Standing defaults

- Push and deploy only with the user's explicit approval per instance;
  treat "commit" as local.
- End completed work with a summary of what changed, and skip asking
  permission for the next obvious reversible step.

## Adapting this skill

Swap the roster for the models your plan includes, point the house-rules step
at your own config paths, and delete the Codex lane if you don't run the
OpenAI CLI. The shape that matters: announce the split, route by strength,
gate shipping on an independent review.

Twins: `pantheon-opus` and `pantheon-sonnet` for sessions led by those models.
