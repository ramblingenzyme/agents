# Working preferences

## Codebase as object

A codebase has two audiences. The machine, which only needs correctness. And the human, who needs to hold the thing in their head. Neither excuses the other. Code that runs but can't be read is broken — it just fails on a different axis than code that doesn't compile.

Treat the codebase as what it is: an aesthetic object. Form isn't decoration sitting on top of working logic — it's where the code's meaning actually lives for the person reading it. Structure, naming, comments aren't documentation of the "real" content. They *are* the content, same rank as the logic itself.

Three surfaces carry this: code, comments, filesystem. A well-placed file, a name that earns its keep, a comment that says what the code can't — same move, different material. Right information in the right layer. Nothing duplicated across layers that already say it. Get the form wrong and the object fails on its own terms, whatever the tests say.

Everything below is that idea applied to a specific moment: building, deciding, verifying, reporting.

## Code philosophy — semantic compression

Efficient doesn't mean shorter. It means the total human cost of a piece of code across its whole life — writing it, debugging it, modifying it, bending it to a new use, plus whatever extra work other code has to do just to talk to it. Compression is in service of that total, not an aesthetic preference for terseness.

Write the concrete version first. No parameters for cases that don't exist. No base class for a hierarchy that isn't there yet. Structure gets revealed by real duplication, not guessed at up front — don't build Employee/Manager/Contractor before any code needs to tell them apart.

Don't extract a shared abstraction until the same logic shows up twice, in real working code. One instance always stays inline until then.

When extraction time comes, do it in steps you can check individually, not one big redesign:

1. Group the data that's already traveling together — the variables getting passed around in both places — into one holder. No new behavior. Just naming what was already moving together.
2. Pull the repeated operations out one at a time. Extract one, confirm nothing broke, extract the next. Behavior changes mid-step means a bug snuck in, not that the refactor got ambitious.

New code needs something already compressed: use it as-is if it fits, adjust it if it's close, add a layer if it's genuinely different. Don't build a second version next to the first.

The test: does the result read like the minimum needed to say what's unique about this case? That's the whole idea of semantic compression — meaning per line, not characters per line. Boilerplate still sitting inline next to the new shared code means the job's half done.

## Filesystem structure and naming

Decide where a file lives on purpose. Don't drop new code into whatever file happens to be open or is convenient.

Check whether the thing belongs in an existing file before creating a new one. A new file needs a real boundary behind it — a distinct concern, a distinct duplication point. "Felt like enough code to split out" isn't one.

Match the project's existing conventions exactly. Files, folders, variable names, all of it. Even the ones you'd have done differently. If the convention's genuinely unclear, ask. Don't invent your own.

## File-level readability

- **Order mirrors how you'd explain it out loud**, not the order it got written in. Orchestration up top, details further down. A reader should be able to stop once they've gone deep enough.
- **A file is one coherent thing.** No real reason to read two parts together means it's not one file's worth of content — doesn't matter how short it is. A tight file mixing two unrelated jobs is worse than a long one doing a single job well.
- **Length is a symptom.** Split when a file's doing two jobs, not when it crosses a line count. Don't keep it long just because it's under some threshold either.
- **Imports at the top are a manifest.** They tell a reader what this file needs from the world before they hit a line of logic. An unused import, or one pulled in for a single incidental call, is noise on that signal.

## Comments

A comment is for what the code can't say on its own — the reasoning, the tradeoff, the constraint, the path not taken. If a comment explains what the code *does*, the code failed, not the comment. Fix it with a better name or an extracted function. Don't bolt a sentence on top. A comment lives in the layer underneath the code. It's not there to sit beside it saying the same thing twice.

The test: would someone reading just the code end up guessing this? Yes — write it down. No — the comment's noise, and noise is worse than silence, because it goes stale and starts lying about code that's moved on without it.

- **Signatures:** comment only when the name and types leave something out.
- **Magic numbers:** name the constant for what it is, comment for why this value and not another.
- **Complex logic:** one comment above the block for the approach. Not a line-by-line narration.
- **TODOs:** leave one wherever a corner got cut, sitting where the next person will actually see it. Not buried in a commit message.
- **Commented-out code:** delete it. Git remembers so the file doesn't have to.

## Scope
An oversized diff is usually what compression-without-evidence looks like at the level of a single change: fixing things preemptively, touching code that wasn't asked about, expanding a task because it seemed related — the same overreach that speculative abstraction is, just applied to scope instead of structure.

- Minimal diff, every time.
- Fix something adjacent only if it shrinks the diff or simplifies the code. Not because you happened to notice it.
- Don't expand past what was asked without saying so first.

## Dependencies

A library doesn't just add a function. It drags in someone else's naming conventions and idioms wholesale — form this codebase never earned through its own repetition. Ask before adding one. No exceptions for "it's small."

## Ambiguity / judgment calls

A wrong guess costs rework. So don't guess. Real judgment call, stop and ask — don't run on an assumption and hope.

## Testing and verification

- Integration and e2e over unit tests. Prove the flow works, not that every function works alone.
- Run tests and build before calling anything done.
- No tests in the area you touched? Say that outright. Don't report "done" with nothing behind it.

## Presenting changes (iterative loop)

No diffs by default. Say what changed clearly enough that opening the file is optional: name the function, name the file, name the actual change.

- Bad: "Fixed the bug in the handler."
- Good: "Changed the timeout check in `fetchUser` from `>` to `>=`. It was letting exactly-at-limit requests through."

A change that genuinely can't be described in one line without ambiguity — multiple files, ripple effects that aren't obvious — earns a diff for that piece. That's the exception, not the default.

Mid-loop replies stay as terse as the first. Report what changed since last message, not the whole thing again.

## Git

No git commands — commit, push, branch, whatever — unless asked. Committing is mine.
