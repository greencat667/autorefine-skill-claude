# Autorefine

A skill for [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) and [Cowork](https://claude.ai) that reviews conversations to find what worked, what didn't, and what could be better — then generates testable optimization hypotheses for your other skills.

Think of it as [autoresearch](https://github.com/karpathy/autoresearch) for your AI workflow. The core pattern is **observe → hypothesize → experiment → keep or discard**, applied to the skills and prompts you use every day.

## What it does

Autorefine is a meta-improvement loop. You point it at a past conversation, and it:

1. **Observes** — reads the transcript, tracks which skills were used, identifies friction points (user corrections, wasted tool calls, unnecessary verbosity, repeated work, quality issues)
2. **Hypothesizes** — turns each friction point into a specific, testable change proposal with predicted impact, confidence level, and risk assessment
3. **Experiments** (opt-in) — tests approved changes by running modified skills against real scenarios and measuring the difference

Over time, it builds a persistent log of what's been tried, what worked, and — critically — what didn't. This prevents re-testing failed ideas and surfaces cross-skill patterns: systemic issues that affect multiple skills at once.

### Cross-skill patterns are the highest-value output

A single systemic fix that improves 10 skills is worth more than 10 skill-specific tweaks. Autorefine tracks these automatically after 3+ reviews. Examples from real use:

- "Skills with built-in checkpoints don't need TodoWrite mirroring those same checkpoints" — saved ~3–4k tokens per session across 3 skills
- "AI fabricates ~45% of citations when drafting without a research step" — led to a global citation rule
- "Multi-round simulation skills need an explicit interactive/batch mode choice" — applied to 2 skills, now a default for new ones

## Installation

**Ask Claude to set it up for you.** If you're using Claude Code or Claude Cowork, you can just say something like *"install the autorefine skill from github.com/greencat667/autorefine-skill-claude"* and Claude will clone the repo and put it in the right place — you don't need to do this by hand.

Or do it yourself: copy the `autorefine/` folder into your project's `.claude/skills/` directory:

```bash
git clone https://github.com/YOUR_USERNAME/autorefine.git
cp -r autorefine/autorefine/ your-project/.claude/skills/autorefine/
```

Claude will pick it up automatically from `available_skills` next time you start a session. No setup script or configuration needed — on first run, autorefine will ask where you'd like to keep your log file and create it for you.

## How it works in practice

Say one of these things to Claude:

| What you say | What happens |
|---|---|
| `autorefine` | Shows recent sessions, asks which to review |
| `review this conversation` | Runs Phase 1 + 2 on the current session |
| `autorefine the trend-report skill` | Finds recent sessions using that skill, reviews 2–3 of them |
| `what could be improved?` | Same as "review this conversation" |
| `run autorefine on [session ID]` | Reviews a specific past session |

Phase 1 (diagnostic) and Phase 2 (hypotheses) always run. Phase 3 (experiments) is opt-in — you choose which hypotheses to test.

### The three phases

**Phase 1 — Observe.** Reads the conversation transcript. Tracks which skills were invoked, how many tool calls each made, where the user had to correct Claude, where work was repeated or wasted. Produces a structured diagnostic report.

**Phase 2 — Hypothesize.** Turns each friction point into a specific, testable proposal. Every hypothesis includes a predicted impact, a confidence level, a risk assessment, and a counter-argument (the sceptic pass). The default bias is "remove something" — leaner skills are almost always better.

**Phase 3 — Experiment (opt-in).** Tests approved hypotheses by running modified skills against real scenarios using subagents. Measures token usage, speed, and output quality against a baseline. Results are logged — including failures.

## Requirements

Autorefine needs access to conversation transcripts:

- **Cowork**: Uses the `list_sessions` and `read_transcript` MCP tools (available by default)
- **Claude Code**: Works with session history via the CLI
- **Phase 3 experiments**: Require subagent support (available in Cowork and Claude Code, not Claude.ai web)

The skill itself is a single Markdown file with no code dependencies.

## The autorefine log

The log is the skill's long-term memory. Every experiment — including failures — gets recorded:

```markdown
## [Date] — [Skill name] — [Hypothesis title]
**Change:** [What was tried]
**Result:** KEPT / DISCARDED / ITERATED
**Impact:** [Token/speed/quality delta]
**Learning:** [One sentence — what this teaches us about skill design]
```

Cross-skill patterns are logged separately:

```markdown
**Cross-skill pattern — [name]:** [Description]
**Seen in:** [List of skills]
**Suggested systemic fix:** [What to change]
```

See [`examples/autorefine-log-example.md`](examples/autorefine-log-example.md) for a full example drawn from real sessions.

## Repository structure

```
autorefine/
├── autorefine/
│   └── SKILL.md                       # Copy this folder to .claude/skills/
├── examples/
│   └── autorefine-log-example.md      # Example log from real use
├── README.md
├── CONTRIBUTING.md
└── LICENSE
```

## Design philosophy

- **Default hypothesis: remove something.** The instinct to add instructions, guardrails, and checks usually makes things worse.
- **Include the counter-argument.** Every hypothesis requires a risk assessment — no optimism without scrutiny.
- **One change per hypothesis.** No bundling. Isolate what works.
- **Log failures.** Negative results are as valuable as positive ones.
- **Human in the loop.** Phase 3 is always opt-in. You're the real judge.

## Using this with another AI assistant

Nothing here is Claude-specific — `autorefine/SKILL.md` is a plain instruction file, no code dependencies. To use it with ChatGPT or another AI assistant, give it this repo's URL (or paste in `SKILL.md`) and ask it to set itself up and review a conversation for you. It's a one-shot request rather than a scheduled one, but the "Requirements" section above is the part to translate: the Claude-side tools that fetch past sessions (`list_sessions`, `read_transcript`) won't exist elsewhere, so on another assistant you'll need to give it an equivalent — e.g. paste in or upload the exported transcript you want reviewed — and Phase 3's subagent experiments will need that assistant's own equivalent of running a modified prompt against test scenarios, if it has one.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) — note this repo isn't actively maintained, so response times on issues and PRs will be slow to nonexistent.

## License

MIT — see [LICENSE](LICENSE).
