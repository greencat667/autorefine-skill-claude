# Autorefine Log

> Persistent log of all skill optimization experiments — including failures.
> Read this before generating new hypotheses to avoid re-testing and to spot cross-skill patterns.

---

## 2026-03-24 — story-writing-skill — H1: Reduce TodoWrite overhead in creative sessions
**Change:** Added guidance: use TodoWrite only for session-level planning, not per-step per-story tracking.
**Result:** KEPT
**Impact:** Estimated −30 TodoWrite calls (~3-4k tokens) in a 3-story session
**Learning:** Skills with built-in AskUserQuestion checkpoints already communicate progress. TodoWrite on top is redundant noise.

## 2026-03-24 — story-writing-skill — H3: Composition checklist beats comprehensive guide
**Change:** Replaced "re-read the full editing guide before writing" with a focused 5-item checklist of the most common failures. Full guide reserved for the review pass.
**Result:** KEPT (iterated from earlier version)
**Impact:** Previous instruction didn't prevent 17 review findings — the writing agent can't hold 22 checks in working memory. Five focused checks are more actionable during composition.
**Learning:** When an AI agent must hold quality constraints in working memory while performing a creative task, a short prioritised checklist (5 items) outperforms a comprehensive reference (22 items). The comprehensive reference is still valuable — but for review, not composition.

## 2026-03-24 — story-writing-skill — H4: Targeted reads for review fixes
**Change:** Updated review step to specify using offset/limit reads when applying fixes, not full-file re-reads.
**Result:** KEPT
**Impact:** Estimated −8-12k tokens per session (3-4 fewer full reads at ~3-4k tokens each)
**Learning:** After the first full read, every subsequent read should be targeted. Review citations provide enough context.

**Cross-skill pattern — TodoWrite overuse in structured workflows:** Skills with built-in checkpoint patterns (AskUserQuestion at decision points) don't need TodoWrite mirroring those same checkpoints. Seen in: story-writing, strategic-wargame (round-by-round pattern). Consider adding to CLAUDE.md: "When a skill has its own progress structure, use TodoWrite sparingly."

---

## 2026-03-24 — research-skill — H1: Hard-stop rule for unautomatable sites
**Change:** After 2 failed automated attempts on certain research sites, stop and tell user to search manually.
**Result:** KEPT (project-level)
**Impact:** Would have saved ~35 tool calls and prevented one context reset
**Learning:** Some sites are structurally unautomatable. A 2-attempt limit is the right discipline. Domain constraints belong in project docs, not global config.

## 2026-03-24 — research-skill — H2: Cache URL patterns for known sites
**Change:** Created a working doc capturing URL/POST formats for frequently-used research sites.
**Result:** KEPT (project-level)
**Impact:** Saves 6–10 discovery tool calls per future session
**Learning:** Rediscovering API/URL patterns is expensive. A "how to search" doc in the project folder pays for itself on the second session.

## 2026-03-24 — system-level — Never construct URLs from memory
**Change:** Added global instruction: "When producing documents with inline citations, NEVER construct a URL from memory or by guessing a path structure. Every URL must come from a WebSearch result, a WebFetch redirect, or a URL the user provided."
**Result:** KEPT (global)
**Impact:** Prevents fabricated citations — observed at ~45% fabrication rate in a research document session
**Learning:** URL construction from memory is pattern-matching, not research. The AI has no reliable knowledge of specific page paths, only domains. The fix is a hard rule, not a softer "try to verify" instruction.

**Cross-skill pattern — fabricated citations in research documents:** AI generates plausible-looking URLs at high fabrication rates when drafting cited documents without a research step. Likely risk in any skill that produces cited reports. Mitigation: global citation rule.

---

## 2026-03-26 — simulation-skill — H1: Configurable output path
**Change:** Replaced hardcoded output path with guidance to use the relevant project's outputs folder.
**Result:** KEPT
**Impact:** Eliminates manual override on every invocation outside the skill's development project.
**Learning:** Output paths should never be hardcoded to a skill's development folder.

## 2026-03-26 — simulation-skill — H2: Heavy output formats opt-in only
**Change:** Changed .docx output from mandatory to "on request only." Default outputs reduced from 5 to 4.
**Result:** KEPT
**Impact:** Saves ~3-5k tokens and ~30s when .docx isn't needed.
**Learning:** Heavy output formats (docx from template) should be opt-in, not default. HTML report covers most sharing needs.

## 2026-03-26 — simulation-skill — H4: Mandatory parallelisation for round 1
**Change:** Added explicit instruction: launch 2 agents in parallel (5 items each), do not write responses in main conversation.
**Result:** KEPT
**Impact:** Saves ~3k main-context tokens and ~30s.
**Learning:** "Use subagents to parallelise" is too soft — it gets ignored. "Parallelisation is mandatory" with a specific split is enforceable.

**Cross-skill pattern — output path hardcoding:** Skills should never hardcode output paths to their development folder. Check all skills with prescribed output paths.

**Cross-skill pattern — interactive vs batch mode:** Multi-round simulation skills both needed an explicit interactive/batch mode choice. When all rounds are delegated to one agent, interactive features are unavailable. If building a new multi-round skill, include this from the start.

**Cross-skill pattern — project knowledge capture:** When a session hits a domain constraint costing ≥10 tool calls to discover, write a working doc so the next session doesn't repeat the discovery.
