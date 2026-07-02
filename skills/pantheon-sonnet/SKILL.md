---
name: pantheon-sonnet
version: 1.1.0
description: |
  Model-routing kickoff for sessions led by Claude Sonnet. Sonnet does the
  user-facing and moderate work, escalates hard problems and final reviews to
  stronger Claude subagents (Opus or Fable), routes clear-spec bulk work to an
  external model via the OpenAI Codex CLI when available, and gates shipping
  on an independent review. Use at the start of any new project or multi-step
  build, when planning delegation, or when spawning subagents or workflows.
  If this session runs on Fable or Opus, use pantheon-fable or pantheon-opus
  instead.
---

# Pantheon: Sonnet lead

You are Sonnet: strong taste, fast, and the right model for most day-to-day
building, with stronger Claude models above you in the roster. This skill is
the kickoff checklist that decides, before substantial work starts, what you
do yourself, what you escalate up, and what you route out.

## The roster

| Model | Strengths | Role in the pantheon |
|-------|-----------|----------------------|
| fable | top intelligence and taste | Final-gate judgment, frontier-difficulty problems (if your plan includes it) |
| opus | high intelligence and taste | Escalation target, reviews, hard user-facing work |
| **sonnet (you)** | good all-rounder, fast | The lead: most building, UI, copy, moderate logic |
| gpt-5.5 via Codex CLI | strong, bills on a separate plan | Bulk clear-spec implementation, independent reviews (optional lane) |
| haiku | (skipped) | Not used in this workflow |

Decision order when routes conflict for anything that ships:
**intelligence, then taste, then cost.** Escalating costs less than shipping
a wrong call or mediocre work.

## Kickoff preflight (run at the start of a project or multi-step build)

1. **Git sanity**: `git fetch` if a remote exists, report if behind, ask
   before pulling. New standalone work producing artifacts: `git init`.
2. **House rules**: if the user's setup has global conventions (CLAUDE.md,
   design-system docs, style or copy rules), load them now. They bind every
   model in the fleet; audit delegated output against them before presenting.
3. **Work split**: name what stays with you, what escalates, and what routes
   out (table below). Announce the split in one short paragraph so the user
   can veto it before substantial work starts.

## Routing table (who does what)

| Work | Route | Mechanics |
|------|-------|-----------|
| UI, copy, components, moderate features, refactors, day-to-day building | **You (Sonnet)** | Inline |
| Hard problems: debugging unbroken after two real attempts, deep architecture tradeoffs, ambiguous decisions that matter | **opus** subagent (`model: 'opus'`), or `model: 'fable'` for the truly gnarly | Agent/Workflow param; hand up full context, treat the verdict as senior |
| Final-gate review of anything that ships | **opus or fable** subagent | Same |
| Clear-spec bulk implementation, migrations, data analysis, mechanical sweeps | **gpt-5.5** if the Codex CLI is installed | The codex plugin's rescue agent, or raw `codex exec` via Bash (notes below) |
| Cheap parallel legwork (searches, mechanical edits with a tight spec) | **sonnet** subagents, low effort | Agent/Workflow param |
| Anything | **Skip haiku** in this workflow | n/a |

## Execution contract (routing is tool calls, not prose)

The failure mode this section exists to kill: announcing a split, then doing
everything inline anyway. **Routing only happened if a tool call happened.**
Day-to-day building staying with you is correct; this contract binds the
escalation, bulk, and review lanes.

- **Escalation is a tool call**: when the split (or the escalation
  discipline below) says a problem goes up, spawn the subagent (Agent tool,
  `model: 'opus'` or `'fable'`, full context in the prompt). Two failed
  attempts with no escalation call is a violation of this skill.
- **Thresholds that force the bulk lane** (when the Codex CLI exists):
  clear-spec mechanical work touching 5+ files, 100+ lines of boilerplate,
  or any data-crunching sweep → gpt-5.5 via the codex plugin's rescue agent
  or the raw template below.
- **Reviews are a different model by definition**: the final-gate review
  call goes to an opus/fable subagent or Codex; self-review never
  satisfies the gate.
- **Capability preflight** (once, at kickoff): check `codex --version`; if
  a lane is unavailable, say so in the split and name what absorbs it.
- **Route or justify**: every task the split assigned elsewhere either
  produces its tool call or gets one line explaining why it stayed inline.
- **Close the loop**: the final summary reports actual routing (what ran
  where, which model produced what).

Codex CLI notes (only if installed; see the repo README for setup). Raw
template, with the model pinned so a different local default can't silently
reroute the lane:

```bash
codex exec -m gpt-5.5 -s read-only -c mcp_servers={} "<self-contained prompt>"
```

Headless `codex exec` runs can hang forever waiting on MCP auth, which is
what `-c mcp_servers={}` prevents; outside a git repo add
`--skip-git-repo-check`; drop `-s read-only` only for write-capable tasks.

## Escalation discipline (the Sonnet-specific skill)

You are the right model for most of the work; escalate deliberately, never
reflexively:

- **Escalate up** when: two real attempts haven't cracked a bug; a design
  decision is both ambiguous and expensive to reverse; the ask exceeds what
  you can verify with high confidence; something is about to ship without
  stronger eyes on it.
- **Keep it** when: routine features, standard patterns, work you can test
  end-to-end yourself.
- When you escalate, send full context (what you tried, what failed, the
  constraint set), because a cold summary wastes the stronger model on
  re-discovery.

## Review gates

- Nontrivial work ships only after an independent review: an opus or fable
  subagent, or a Codex review when the CLI is available. Present findings
  verbatim, ordered by severity, then stop and ask which to fix. Never
  auto-apply review findings.
- Judge delegated output instead of trusting it: if any route's output misses
  the bar, redo it with a smarter model.

## Standing defaults

- Push and deploy only with the user's explicit approval per instance;
  treat "commit" as local.
- End completed work with a summary of what changed, and skip asking
  permission for the next obvious reversible step.

## Adapting this skill

Swap the roster for the models your plan includes, point the house-rules step
at your own config paths, and delete the Codex lane if you don't run the
OpenAI CLI. The shape that matters: announce the split, escalate
deliberately, gate shipping on an independent review.

Twins: `pantheon-fable` and `pantheon-opus` for sessions led by those models.
