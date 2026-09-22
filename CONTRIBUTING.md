# Contributing to Autorefine

Thanks for your interest in improving autorefine.

**This repo is not actively maintained.** It was published as a one-time release, not something
I'm committed to reviewing or updating on an ongoing basis. Issues and PRs may sit unanswered for
a long time, or indefinitely. If you need changes, forking is likely faster than waiting on a
response here.

Here's how to contribute anyway, if you'd still like to.

## What's most useful

The highest-value contributions are **cross-skill patterns** — things you've noticed while using autorefine on your own skills that could help others. If you've found a pattern that improved multiple skills at once, that's gold.

Examples of good contributions:

- A new cross-skill pattern you've observed (with evidence from your autorefine log)
- A refinement to the hypothesis format that makes hypotheses more actionable
- A fix to the Phase 1 diagnostic that catches a class of friction it currently misses
- Improvements to the example log that make the format clearer for new users

## How to contribute

1. Fork the repository
2. Create a branch (`git checkout -b pattern/your-pattern-name`)
3. Make your changes
4. Open a pull request with a clear description of what you changed and why

For cross-skill patterns, include:

- The pattern description
- Which skills you observed it in
- What the fix was
- What the measured or estimated impact was

## What to avoid

- Changes that make the SKILL.md longer without clear justification (the default hypothesis is "remove something")
- Adding dependencies — the skill is a single Markdown file with no code deps, and should stay that way
- Changes specific to a single user's workflow — keep it general

## Questions?

Open an issue if you like, but see the note above — don't expect a quick reply.
