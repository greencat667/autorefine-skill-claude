---
name: autorefine
description: >
  Review a Cowork conversation to find friction, waste, and quality issues in the skills and prompts used — then generate and optionally test optimization hypotheses. Use this skill whenever someone says "review this conversation", "autorefine", "refine this skill", "what could be improved", "optimize this skill", "review the session", "what went wrong", "why was that slow", "how could that be better", "skill review", "conversation review", "prompt optimization", or "find inefficiencies". Also use when someone pastes a session ID and asks for analysis, or when they say "run autorefine on [skill name]". This skill is the meta-improvement loop for all other skills — think of it as autoresearch for Cowork.
---

# Autorefine: Conversation Review & Skill Optimization

A skill that reviews Cowork conversations to find what worked, what didn't, and what could be better — then optionally tests improvements in an automated loop.

Inspired by Karpathy's autoresearch: the core pattern is **observe → hypothesize → experiment → keep or discard**. The difference is that our "training runs" are skill invocations, and our "loss function" is a mix of token efficiency, execution speed, output quality, and user satisfaction.

## How this skill works

There are three phases. Phase 1 always runs. Phase 2 always runs (it's fast and cheap). Phase 3 is opt-in — the user decides which hypotheses to test.

---

## Phase 1 — Observe

**Goal:** Build a structured diagnostic of what happened in a conversation.

### Step 1: Get the transcript

Ask the user which conversation to review. Three options:

1. **"This conversation"** — use `list_sessions` to find the current session, then `read_transcript` with a high limit (50–100 messages) to get the full history.
2. **A specific session** — the user provides a session name or ID. Use `list_sessions` to find it, then `read_transcript`.
3. **A specific skill** — the user names a skill they want to optimize. Use `list_sessions` to find recent sessions, read transcripts from 2–3 of them, and focus analysis on invocations of that skill.

If the user just says "autorefine" without specifying, show them the 5 most recent sessions and ask which one.

### Step 2: Analyse the transcript

Read the full transcript. For each segment of the conversation, track:

**Skill invocations:**
- Which skills were triggered (look for Skill tool calls and SKILL.md reads)
- How many tool calls each invocation made
- Whether the skill completed its task or the user had to intervene/correct
- Whether the skill asked unnecessary clarifying questions when it had enough context
- Whether the skill produced output the user actually used vs output that was discarded or redone

**Friction points:**
- Places where the user corrected Claude's approach
- Places where Claude repeated work or went down a dead end
- Places where Claude was verbose (long explanations before acting)
- Places where Claude asked for information it already had (in memory files, earlier in conversation, or in the skill itself)
- Tool call failures or retries
- Unnecessary file reads (reading files that weren't used)

**Token signals:**
- Long assistant messages that could have been shorter
- Repeated context loading (reading the same file multiple times)
- Overly detailed tool call descriptions
- Preamble before action ("Let me think about this..." / "Great question! I'll...")

**Quality signals:**
- Did the final output match what the user asked for on the first attempt?
- How many revision rounds were needed?
- Were there any factual errors or misunderstandings?
- Did the user express satisfaction or frustration?

### Step 3: Produce the diagnostic report

Output a structured report with these sections:

```
## Conversation Review: [session name]

### Summary
[2-3 sentences: what was attempted, what was achieved, overall assessment]

### Skills Used
| Skill | Invocations | Tool Calls | Corrections | Assessment |
|-------|------------|------------|-------------|------------|
| ...   | ...        | ...        | ...         | ...        |

### Friction Points
[Numbered list. Each entry: what happened, where in the conversation, estimated impact (token waste / time waste / quality issue)]

### Token Efficiency Observations
[What was wasteful, with rough estimates where possible]

### What Worked Well
[Don't just list problems — note what was effective, fast, or well-received]

### Phase 2 Ready
[One line: "X friction points identified. Ready to generate hypotheses."]
```

Save this report to a consistent location — ideally a `reports/` folder within the autorefine project directory, or wherever the user keeps autorefine outputs. Ask on first run if unsure. Never save to the workspace root.

---

## Phase 2 — Hypothesize

**Goal:** Turn each friction point into a specific, testable optimization hypothesis.

### Step 1: Read the relevant skill source(s)

For each skill identified in Phase 1, read its SKILL.md. Also read any referenced files (scripts, reference docs) that are relevant to the friction points found.

### Step 2: Generate hypotheses

For each friction point, write a hypothesis in this format:

```
### Hypothesis [N]: [Short title]

**Friction point:** [Which friction point from Phase 1 this addresses]
**Skill affected:** [Skill name, or "conversation-level" if it's about Claude's general behaviour]
**Proposed change:** [Specific, concrete change — e.g. "Add a stop condition after Step 3 that checks whether the user already provided X before researching it" or "Remove the preamble paragraph in the output template" or "Replace the 5-step research phase with a 2-step version when the user provides source material"]
**Predicted impact:**
- Token usage: [increase/decrease/neutral, with rough estimate if possible]
- Speed: [faster/slower/neutral]
- Output quality: [better/worse/neutral, and why]
**Confidence:** [high/medium/low — how sure are you this will help?]
**Risk:** [What could go wrong? Could this break other use cases?]
**Testable?** [Yes — describe test scenario / No — explain why, suggest manual review instead]
```

Important principles for good hypotheses:

- **Be specific.** "Make the skill faster" is not a hypothesis. "Remove the redundant file-existence check in Step 2, which adds ~3 tool calls when the file always exists" is.
- **One change per hypothesis.** Don't bundle multiple changes — you need to isolate what works.
- **Include the counter-argument.** Every optimization has a trade-off. State it explicitly. This is the sceptic pass.
- **Distinguish skill-level from system-level.** Some friction comes from a specific skill's instructions. Some comes from how Claude generally behaves (preamble, over-asking, context repetition). Label which is which — system-level observations get logged for cross-skill pattern detection.

### Step 3: Present and prioritise

Present all hypotheses to the user. Suggest a priority order based on:
1. Estimated impact (biggest improvement first)
2. Confidence (high-confidence wins over speculative)
3. Risk (low-risk wins over potentially-breaking changes)

Ask: **"Which of these do you want to test? Or should I apply the high-confidence ones and we'll review?"**

If the user says "just apply the obvious ones" — apply any hypothesis marked high-confidence and low-risk directly, and note what changed in the autorefine log.

---

## Phase 3 — Experiment (opt-in)

**Goal:** Test approved hypotheses by running the modified skill against scenarios and measuring the difference.

### Step 1: Establish baseline

Before modifying anything:
- Copy the current skill to a working directory: `[project-folder]/working/skill-snapshot/`
- Define 2–3 test scenarios. Sources for scenarios:
  - Extract from the transcript (use the actual prompts the user gave)
  - Ask the user for representative scenarios
  - Generate synthetic ones based on the skill's description
- Run the **unmodified** skill against each scenario using a subagent. Record:
  - Token count (from the subagent completion notification)
  - Duration
  - Output (saved to `[project-folder]/working/baseline/`)

### Step 2: Apply and test each hypothesis

For each approved hypothesis:
1. Fork the skill from the snapshot
2. Apply the single proposed change
3. Run against the same test scenarios using a subagent
4. Record token count, duration, output
5. Compare against baseline

### Step 3: Score and report

For each hypothesis tested, produce:

```
### Result: [Hypothesis title]

**Change applied:** [What was changed in the SKILL.md]
**Token delta:** [+/- N tokens, % change]
**Duration delta:** [+/- N seconds, % change]
**Quality assessment:** [Same / Better / Worse — with explanation]
**Verdict:** KEEP / DISCARD / ITERATE
**Evidence:** [Specific observations from the test outputs]
```

### Step 4: Log results

**This is critical.** Every experiment — including failures — gets logged to a persistent file.

By default, the log lives at `autorefine-log.md` in the user's working directory (or wherever the user prefers — ask on first run if no log exists yet, and remember the location).

Format:

```
## [Date] — [Skill name] — [Hypothesis title]
**Change:** [What was tried]
**Result:** KEPT / DISCARDED / ITERATED
**Impact:** [Token/speed/quality delta]
**Learning:** [One sentence — what this teaches us about skill design]
```

This log accumulates over time. When generating new hypotheses (Phase 2), always read this log first to avoid re-testing things that have already been tried, and to spot cross-skill patterns.

### Step 5: Apply kept changes

For any hypothesis marked KEEP:
1. Apply the change to the live skill
2. Confirm with the user before overwriting
3. Note the change in the skill's project log if one exists

---

## Cross-skill pattern detection

After reviewing 3+ conversations (check the autorefine log for history), look for patterns that recur across different skills:

- "All skills spend too many tokens on file discovery"
- "Skills consistently over-explain before acting"
- "Skills don't check memory files before asking clarifying questions"
- "Preamble/postamble is consistently trimmed by the user"

When you spot a cross-skill pattern, flag it clearly:

```
**Cross-skill pattern detected:** [Description]
**Seen in:** [List of skills]
**Suggested systemic fix:** [What to change — could be a CLAUDE.md instruction, a shared convention, or a change applied to multiple skills]
```

These are the highest-value findings. A single systemic fix that improves 10 skills is worth more than 10 skill-specific tweaks.

---

## Output formats

- **Phase 1 report:** Markdown file saved to workspace + shown in conversation
- **Phase 2 hypotheses:** Shown in conversation (not saved separately — they're either tested or discarded)
- **Phase 3 results:** Markdown file saved to workspace + entries added to the autorefine log
- **Cross-skill patterns:** Flagged in conversation + logged to autorefine log

---

## What this skill is NOT

- It's not a replacement for human judgment on output quality. Quality scoring in Phase 3 is approximate — the user is the real judge.
- It's not a continuous background agent (yet). It runs when invoked. The aspiration is autonomous between-session optimization, but this is a structured human-in-the-loop tool.
- It's not about making skills longer or more detailed. The default hypothesis should be "remove something" not "add something." Leaner skills are almost always better.

---

## Quick start examples

**"Autorefine the persona panel"** → Find 2–3 recent sessions that used persona-panel. Run Phase 1 on each. Aggregate friction points. Phase 2: hypotheses specific to persona-panel. Phase 3 if the user opts in.

**"Review this conversation"** → Phase 1 on the current session. Phase 2 with hypotheses. Ask if they want Phase 3.

**"What could be improved about the trend report skill?"** → Same as the first example but for trend-report.

**"Run autorefine"** → Show recent sessions, ask which one(s) to review.
