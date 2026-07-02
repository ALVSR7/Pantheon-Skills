# Pantheon

Model-routing skills for [Claude Code](https://claude.com/claude-code). One idea: your lead model's tokens are the scarcest resource in your setup, so spend them on judgment and route everything else to the model that's actually right for it.

Pantheon ships three kickoff skills, one per lead model. Each is a checklist your session runs at the start of a project or multi-step build: announce the work split, route by strength, escalate deliberately, and gate anything that ships on an independent review.

## Why this saves tokens (and produces better work)

Running everything on your strongest model burns your subscription on work a cheaper model does just as well. Running everything on a cheap model ships mediocre judgment. The fix is a roster:

| Model | Best at | Worst use of it |
|-------|---------|-----------------|
| Fable | judgment, architecture, taste, hard debugging | grinding through a 40-file mechanical migration |
| Opus | user-facing work, reviews, most architecture | bulk boilerplate |
| Sonnet | day-to-day building, parallel legwork | final calls on ambiguous, expensive decisions |
| gpt-5.5 (via the OpenAI Codex CLI) | clear-spec bulk implementation, independent reviews | taste-critical UI and copy |

The Codex lane is the big cost lever: gpt-5.5 bills through your OpenAI plan, so heavy implementation work stops consuming Claude tokens entirely. It also gives you something a same-model review can't: a genuinely independent second opinion from a different frontier model.

Three rules hold the system together:

1. **Intelligence, then taste, then cost.** When routes conflict for anything that ships, pick in that order. Redoing cheap output with a smarter model costs less than shipping the wrong thing.
2. **Announce the split.** Before substantial work starts, the lead model says in one paragraph what it keeps, what it escalates, and what it routes out, so you can veto the plan in five seconds.
3. **Independent review before shipping.** Findings get presented verbatim and the agent stops and asks which to fix. Auto-applied review findings defeat the point of review.

## The three skills

Pick the one matching your session's model. Each knows about the others and tells the agent to switch if it detects a mismatch.

- **`/pantheon-fable`**: the lead does judgment, architecture, and taste work itself; bulk goes to Codex; Opus and Sonnet subagents parallelize.
- **`/pantheon-opus`**: the lead does taste and user-facing work; frontier-difficulty problems escalate up to a Fable subagent when available; bulk goes to Codex.
- **`/pantheon-sonnet`**: the lead does day-to-day building; hard problems and final reviews escalate to Opus or Fable; bulk goes to Codex. Includes the escalation discipline that makes a Sonnet-led setup work.

Every skill degrades gracefully: without the Codex CLI the bulk lane is skipped, and without Fable on your plan the escalation targets fall back to Opus.

## Install

### Option A: copy the skills (simplest)

```bash
git clone https://github.com/ALVSR7/pantheon.git
cp -r pantheon/skills/pantheon-* ~/.claude/skills/
```

Restart Claude Code (or start a new session). Invoke with `/pantheon-fable`, `/pantheon-opus`, or `/pantheon-sonnet`.

### Option B: install as a plugin

```
/plugin marketplace add ALVSR7/pantheon
/plugin install pantheon@pantheon
```

Plugin installs namespace the skills (for example `/pantheon:pantheon-fable`).

### Optional: the Codex lane

The gpt-5.5 routing needs the OpenAI Codex CLI:

```bash
npm install -g @openai/codex
codex login
```

Optionally add the official Codex plugin for Claude Code, which provides the rescue agent and review commands the skills prefer over raw CLI calls:

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
```

One gotcha the skills already know about: headless `codex exec` calls from an agent can hang forever waiting on interactive MCP auth. The skills pass `-c mcp_servers={}` to avoid it; keep that flag if you script your own calls.

### Make it automatic (recommended)

Skills fire when invoked. To get the routing pre-flight on every session without remembering to type it, add a SessionStart hook in `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "jq -n '{hookSpecificOutput:{hookEventName:\"SessionStart\",additionalContext:\"At the start of any new project or multi-step build, invoke the pantheon kickoff skill matching the session model (/pantheon-fable, /pantheon-opus, or /pantheon-sonnet).\"}}'"
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

Built by [ALVSR7](https://github.com/ALVSR7). The workflow was developed and refined in daily use across Claude Fable, Opus, Sonnet, and OpenAI Codex (gpt-5.5), and these skills were authored with Claude Code running Fable, then reviewed by Codex before publishing.

## License

MIT
