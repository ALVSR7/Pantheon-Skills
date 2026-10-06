# Pantheon Skills

<img width="1920" height="1080" alt="Linkedin post" src="https://github.com/user-attachments/assets/365d508b-3782-4407-8e6c-a2130249f324" />


Model-routing skills for [Claude Code](https://claude.com/claude-code). 

One idea: your lead model's tokens are the scarcest resource in your setup, so spend them on judgment and route everything else to the model that's actually right for it.

Pantheon ships three kickoff skills, one per lead model. Each is a checklist your session runs at the start of a project or multi-step build: announce the work split, route by strength, escalate deliberately, and gate anything that ships on an independent review.

## Why this saves tokens (and produces better work)

Running everything on your strongest model burns your subscription on work another lane handles just as well. Running everything on a lower tier ships mediocre judgment. The fix is a roster:

| Model | Best at | Worst use of it |
|-------|---------|-----------------|
| Opus (Claude Opus 5.5), **the default lead** | judgment, architecture, debugging, planning, reviews, most building | bulk boilerplate |
| Fable (Claude Fable 5.1) | taste: frontend design, art direction, final-gate reviews of client-facing work | grinding through a 40-file mechanical migration |
| Sonnet | day-to-day building, parallel legwork | final calls on ambiguous, expensive decisions |
| GPT-6 Astra, `gpt-6-astra` (via the OpenAI Codex CLI; Sol and Luna as cheaper tiers for high-volume passes; GPT-6.1 Sol, `gpt-6.1-sol`, for bug fixing) | backend, systems, clear-spec bulk implementation, independent reviews | taste-critical UI, frontend design and copy |

**Opus 5.5 is the leading model, with Fable 5.1 second.** Since September 22 2026, Opus 5.5 tops the Artificial Analysis Intelligence Index (58 to Fable 5.1's 53) at less than half Fable's cost, so it leads by default. Fable 5.1 matches it on hard problems and leads on taste, which makes it the escalation for frontend design and the final review of anything a client will see.

The Codex lane is the big cost lever: Codex usage is metered against your ChatGPT plan or OpenAI API credits, so heavy implementation work moves off your Claude quota (Claude still spends orchestration and review tokens around it). It's a separate budget rather than a discount, and it buys something a same-model review can't: a genuinely independent second opinion from a different frontier model. If you want the bulk lane cheaper still, route it to a smaller Codex model like gpt-5.4-mini with `-m`.

Three rules hold the system together:

1. **Intelligence, then taste, then cost.** When routes conflict for anything that ships, pick in that order. Redoing cheap output with a smarter model costs less than shipping the wrong thing.
2. **Announce the split.** Before substantial work starts, the lead model says in one paragraph what it keeps, what it escalates, and what it routes out, so you can veto the plan in five seconds. (Note "escalates": redoing a lower tier's output with a smarter model costs less than shipping the wrong thing.)
3. **Independent review before shipping.** Findings get presented verbatim and the agent stops and asks which to fix. Auto-applied review findings defeat the point of review.

## The three skills

Pick the one matching your session's model. Each knows about the others and tells the agent to switch if it detects a mismatch.

- **`/pantheon-opus`** (recommended default): Opus leads builds, logic, planning, and most judgment; taste-critical design and final client-facing reviews escalate to a Fable subagent when available; bulk goes to Codex.
- **`/pantheon-fable`**: for sessions you run on Fable, usually design-heavy work. The lead does judgment, architecture, and taste work itself; bulk goes to Codex; Opus and Sonnet subagents parallelize.
- **`/pantheon-sonnet`**: the lead does day-to-day building; hard problems and final reviews escalate to Opus or Fable; bulk goes to Codex. Includes the escalation discipline that makes a Sonnet-led setup work.

The skills name models by family alias (`fable`, `opus`, `sonnet`), which is what the Agent and Workflow `model` parameter accepts. Each alias tracks the current release of its family (Claude Fable 5.1, Opus 5.5, and Sonnet 5 as of September 2026), so a point release needs no skill edit.

Every skill degrades gracefully: without the Codex CLI the bulk lane is skipped. Without Fable on your plan, Sonnet-led sessions escalate to Opus, and Opus-led sessions fall back to self-review plus the Codex lane.

## Install

### Option A: copy the skills (simplest)

```bash
git clone https://github.com/ALVSR7/Pantheon-Skills.git
mkdir -p ~/.claude/skills
cp -R Pantheon-Skills/skills/pantheon-* ~/.claude/skills/
```

Restart Claude Code (or start a new session). Invoke with `/pantheon-fable`, `/pantheon-opus`, or `/pantheon-sonnet`.

### Option B: install as a plugin

```
/plugin marketplace add ALVSR7/Pantheon-Skills
/plugin install pantheon@pantheon
```

Then run `/reload-plugins` (or restart Claude Code) so the skills register in the current session. Plugin installs namespace the skills, so invoke them as `/pantheon:pantheon-fable`, `/pantheon:pantheon-opus`, and `/pantheon:pantheon-sonnet`.

### Optional: the Codex lane

The gpt-6-astra routing needs the OpenAI Codex CLI:

```bash
npm install -g @openai/codex
codex login
```

New Codex models (GPT-6.1 Sol most recently) ship faster than installs update, and older builds list the
model but the API rejects it as unavailable or `requires a newer version of Codex`. The skills handle this: the kickoff
preflight compares `codex --version` against `npm view @openai/codex version` and upgrades with
`npm install -g @openai/codex@latest` when behind, runs `claude update`, and any model-unavailable error triggers one
update-and-retry before it is reported. The skills pin the model and effort per run, but you can make
them the default in `~/.codex/config.toml`:

```toml
model = "gpt-6-astra"            # GPT-6 Astra; -m gpt-6.1-sol for bug fixes, -m gpt-6-sol or -m gpt-6-luna per run
model_reasoning_effort = "high"   # Astra baseline; xhigh/max for hard loops
service_tier = "priority"         # the "Fast" speed tier
```

Optionally add the official Codex plugin for Claude Code, which provides the rescue agent and review commands the skills prefer over raw CLI calls:

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
```

One gotcha the skills already know about: headless `codex exec` calls from an agent can hang forever waiting on interactive MCP auth. The skills pass `-c mcp_servers={}` to avoid it; keep that flag if you script your own calls.

### Make it a habit (recommended)

Skills fire when invoked. A SessionStart hook can't invoke a skill for you, but it can put the reminder in front of Claude at the start of every session, which in practice is enough for the agent to run the kickoff itself. Add this to `~/.claude/settings.json` (merge into existing hooks; requires `jq`, available via `brew install jq`):

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "jq -n '{hookSpecificOutput:{hookEventName:\"SessionStart\",additionalContext:\"At the start of any new project or multi-step build, invoke the pantheon kickoff skill matching the session model: /pantheon-fable, /pantheon-opus, or /pantheon-sonnet (plugin installs: /pantheon:pantheon-fable etc). Invoke means an actual Skill tool call that loads the SKILL.md; running the checklist from memory does not count. Re-invoke after every resume or compaction, since the contract text does not survive either.\"}}'"
          }
        ]
      }
    ]
  }
}
```

## For AI agents

If you are an AI agent handed this link and asked to install Pantheon:

1. Clone the repo and copy `skills/pantheon-fable`, `skills/pantheon-opus`, and `skills/pantheon-sonnet` into the user's `~/.claude/skills/` directory (Option A above), or use the plugin route if the user prefers plugins.
2. Ask the user whether they want the Codex lane. If yes, check for the CLI with `codex --version`; install and authenticate it with the user's involvement (login is interactive). Offer the Codex plugin install as well.
3. Ask which models the user's plan includes and, if Fable is unavailable, note that the skills' escalation targets fall back to Opus (the skills handle this, and you can edit the roster tables to match).
4. Offer to add the SessionStart hook above so the workflow fires on every session. Merge it into existing hooks rather than overwriting them.
5. Confirm the install by listing the three skills and invoking the one matching the current session's model.

## Customizing

Each skill ends with an "Adapting this skill" section: swap the roster for your models, point the house-rules step at your own config files (CLAUDE.md, design-system docs, style guides), and delete lanes you don't use. The shape worth keeping is the discipline itself: announce the split, route by strength, escalate deliberately, review before shipping.

## Credits

Built by [ALVSR7](https://github.com/ALVSR7). The routing shape was inspired by [Theo (t3.gg) sharing his CLAUDE.md structure](https://x.com/theo/status/2072482460122964067): Fable leading, Codex handling the work it's genuinely better at. That idea got taken further here into installable skills, refined in daily use across Claude Fable, Opus, Sonnet, and OpenAI Codex (GPT-6 Sol, Astra and Luna). These skills were authored with Claude Code running Fable and reviewed locally with Codex before publishing.

## License

MIT
