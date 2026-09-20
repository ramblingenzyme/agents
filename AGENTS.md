# Working preferences

## Codebase as object

A codebase has two audiences — the machine, which only needs correctness, and the human, who needs to be able to hold the thing in their head. These aren't a trade-off to balance against each other; they're two separate, equally real targets, and hitting one doesn't excuse missing the other. Correct-but-illegible code is a failure as surely as elegant-but-wrong code is.

Treat the codebase as the aesthetic object it is: form is not decoration on top of working logic, it's where the code's meaning for a human reader actually lives. Structure, naming, and comments aren't documentation of the "real" content — they *are* content, on equal footing with the logic itself. Get the form wrong and the object fails on its own terms, whatever the tests say.

Code, comments, and filesystem structure/naming are the three surfaces this shows up on. Decisions in one should be made with the others in view, not in isolation — a well-placed file, a name that does its job, and a comment that only says what the code can't are the same move applied to different material: put the right information in the right layer, and don't duplicate information across layers that already carry it. This is why file placement matters as much as what's in the file, why a good name removes the need for a comment, and why an abstraction pulled out too early pollutes both the code's structure and whatever would have to be said about it.

The three sections below — code philosophy, filesystem/naming, comments — are this principle applied to each surface specifically. The workflow rules after them aren't separate concerns bolted on; each one is this same discipline applied to a different moment in the process — building, deciding, verifying, reporting.

## Code philosophy — semantic compression

**What "efficient" means here:** not shorter code, not fewer files — minimizing the total human effort a piece of code costs across its entire lifetime. That includes the time to write it, debug it, modify it, adapt it to new uses, and any extra work done to *other* code just to interface with it. Compression is in service of that total, not an aesthetic preference for terseness.

Write the concrete version first — no parameters, base classes, or interfaces for cases that don't exist yet. Let structure reveal itself from real duplication, not from modeling the domain upfront (e.g. don't build an Employee/Manager/Contractor hierarchy before any code needs to distinguish them).

Don't extract a shared abstraction until the same logic is duplicated twice, in real working code. One instance always stays inline until then.

When it's time to extract, do it in small, independently verifiable steps rather than one redesign:
1. First, group the data that's already traveling together across the duplicated code (the same variables getting passed around or reused in both places) into one shared holder — no new behavior yet, just naming what's already being threaded through together.
2. Then pull the repeated operations on that data out one at a time — extract one, confirm nothing changed, extract the next. Each step should leave behavior identical to before; if behavior changes mid-step, that's a bug, not part of the refactor.

When new code needs something already compressed: use it as-is if it fits, modify it if it's close, add a layer if it's genuinely different. Don't fork a parallel version instead.

Test: the result should read as close to the minimum information needed to express what's unique about this case. Leftover inline boilerplate next to the new shared code means the compression didn't finish.

## Filesystem structure and naming

Where a file lives and what it's called is part of the design, not an afterthought — decide it deliberately, don't default to dropping new code in whatever file/folder is open or convenient.

Before creating a new file, check whether the thing belongs in an existing one first — new files should reflect a real boundary (a distinct concern, a distinct duplication point), not just "this felt like enough code to split out."

Match the existing project's structure and naming conventions exactly — files, folders, and identifiers (variables, functions, types) alike — even if you'd organize it differently starting from scratch. If the convention is genuinely unclear or inconsistent, ask rather than picking your own pattern.

## Comments

A comment exists for what the code can't say for itself — the reasoning, the tradeoff, the constraint, the rejected alternative. If a comment explains what the code does, that's a failure of the code, not a job for the comment: the fix is a better name, an extracted function, clearer types — not a sentence bolted on top. Comments live in the layer underneath the code, not alongside it restating it.

The test before writing one: would someone reading just the code, no comment, eventually have to guess this? If yes, write it down. If the code already says it, the comment is noise — and noise is worse than nothing, because it goes stale and starts lying about code that's since changed underneath it.

This is a discipline applied with judgment, not a checklist to satisfy mechanically — the specifics below are where that judgment tends to land, not a substitute for it:

- Signatures: comment only when name/types don't already make purpose and usage obvious.
- Magic numbers/constants: name the constant, then comment *why this value* — the name covers what it is, the comment covers why it's this and not something else.
- Complex logic: one comment above the block explaining the approach, not narration of each line inside it.
- TODOs: always leave one when cutting a corner — it has to live where the next person touching the code will see it, not just in a change description.
- Commented-out code: delete it when replacing something. Git history is the record.

## Scope

An oversized diff is usually what compression-without-evidence looks like at the level of a single change: fixing things preemptively, touching code that wasn't asked about, expanding a task because it seemed related — the same overreach that speculative abstraction is, just applied to scope instead of structure.

- Minimal diff by default.
- Fix adjacent issues only if doing so simplifies the code or shrinks the diff — not just because you noticed them.
- Don't expand scope beyond what was asked without flagging it first.

## Dependencies

A new library isn't just a functional choice — it imports someone else's naming conventions, idioms, and structure wholesale, form that was never compressed from this codebase's own actual repetition. It's the most direct way outside form gets imposed on the object instead of emerging from use, which is precisely what the compression discipline above is arguing against. Treat it accordingly: never add a new library or package without asking first, regardless of how minor it seems.

## Ambiguity / judgment calls

Guessing on an ambiguous decision is the same overreach as speculative abstraction, aimed at a design choice instead of code structure — acting on an assumption instead of waiting for real evidence of what's actually needed. Don't guess where a wrong guess costs rework: when there's a genuine judgment call, stop and ask.

## Testing and verification

Verification is the machine-audience half of the two-audience premise made concrete — it's where "correct" actually gets checked, as distinct from "legible," which the rest of this document is mostly about. Neither substitutes for the other.

- Favor integration/e2e tests over granular unit tests. Test the flow actually working, not every internal function in isolation.
- Always run tests/build before reporting something done.
- If no tests exist for the area you touched, say so explicitly rather than reporting "done" with no verification.

## Presenting changes (iterative loop)

Without a visible diff, the description of a change *is* its form for the human reader — the same job a comment does for code, applied to the act of reporting a change instead of the change itself. A vague description is exactly as much a failure as a comment that doesn't earn its place.

No diffs by default. Describe what changed, specifically enough that I don't need to open the file to know what happened: name the function/file touched and the actual change, not just "updated the logic."

- Bad: "Fixed the bug in the handler."
- Good: "Changed the timeout check in `fetchUser` from `>` to `>=` — it was letting exactly-at-limit requests through."

If a change is genuinely hard to describe in one line without ambiguity (touches multiple files, non-obvious ripple effects), showing the diff for that specific piece is the exception — flag it as an exception rather than defaulting to it.

Mid-loop replies stay as terse as the first description — don't re-summarize the whole change each round, just what changed since the last message.

## Git

Do not run git commands (commit, push, branch, etc.) unless explicitly asked. Committing is mine to do. This is a division of responsibility, not a form concern — no connection to the rest of this document is implied.
