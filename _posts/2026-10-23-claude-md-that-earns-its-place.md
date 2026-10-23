---
layout: post
title: "CLAUDE.md that earns its place: start with 5 lines, grow by friction"
date: 2026-10-23 08:30:00 +0200
tags: [claude-code, claude-md, memory, java]
---

**⏱ 10 min · Level: beginner**

## 1. The idea (2 min)

Every Claude Code session starts with a fresh context window. `CLAUDE.md` is a markdown file that Claude reads at the start of every session. It is how you stop re-explaining your project.

The common mistake is to write a big CLAUDE.md on day one. Long files cost context and the official docs say they reduce how well Claude follows them. The docs suggest staying **under 200 lines per file**.

A better way: **start with about 5 lines and grow by friction.** Add a line only when one of these happens:

* Claude makes the same mistake a second time.
* A code review catches something Claude should have known about this codebase.
* You type the same correction you typed last session.
* A new teammate would need the same fact to be productive.

Two more points:

* CLAUDE.md is *context*, not enforced configuration. If something must never happen, use a permission rule or a hook (see the lesson from 21 October).
* Write instructions that can be checked. "Run `./mvnw test` before finishing" is good. "Test your changes" is vague.

## 2. Why it matters for developers (1 min)

On a Spring Boot service, Claude can read your code, but it cannot guess things that are not in the code: which Maven command is the real one, that you use Testcontainers and not H2, or that DTOs are Java records. Without a CLAUDE.md you pay for these facts again in every session, in wrong guesses and in corrections.

With a short, current file, the first attempt is closer to what you want. A file stuffed with rarely needed rules does the opposite: the important lines get lost.

## 3. Example (3 min)

A starter `CLAUDE.md` in the root of a Spring Boot repo (5 lines of facts, nothing else):

```markdown
# Orders service

- Build and test: `./mvnw verify`. Single test: `./mvnw -Dtest=OrderServiceTest test`.
- Integration tests use Testcontainers (PostgreSQL). Never use H2.
- Package by feature: `com.example.orders.checkout`, not `controller/service/repository`.
- DTOs are Java records. Entities are never returned from controllers.
- Database changes go through Flyway migrations in `src/main/resources/db/migration`.
```

Now see how it grows by friction. Over three weeks:

* Claude put a new `@Service` in a `service/` package. You add the package-by-feature line (already above).
* A reviewer finds `System.out.println` in a PR. You add: `- Log with SLF4J (@Slf4j). No System.out.`
* You explain the Flyway naming scheme for the third time. You add one line with the pattern.

Notice what does **not** go in:

* Things Claude can read from the code (class lists, obvious Spring basics).
* Multi-step procedures such as "how to release". The docs say these belong in a *skill*.
* Rules for one part of the codebase only. These belong in a path-scoped rule in `.claude/rules/` (a later lesson).

Useful extras from the docs:

* `./CLAUDE.md` or `./.claude/CLAUDE.md` is shared with the team through git. `./CLAUDE.local.md` is for your private notes; put it in `.gitignore`.
* `~/.claude/CLAUDE.md` holds personal preferences for all projects.
* `<!-- maintainer notes -->` HTML comments are stripped before the text is sent to Claude, so they cost no context.
* `@docs/git-instructions.md` imports another file. Be careful: imported files still load together with the CLAUDE.md, so imports organize the file but do not save context.

## 4. Try it (3 min)

1. Open your main project in a terminal and start `claude`. Run `/context` and look at the **Memory files** list. This shows what is loaded now.
2. If there is no CLAUDE.md, create one by hand with the 5-line pattern above, using your real build and test commands. (You can also run `/init` to get a generated draft, then cut it down to facts Claude would not discover itself.)
3. Restart the session and ask Claude: "How do I run a single test here?" Check that the answer matches your file.
4. Look at your last three code review comments. Is one of them a rule Claude could not know? Add it as one concrete line.
5. If your file has grown, run `/doctor prompt-audit`. Claude reports outdated, missing or conflicting instructions and proposes edits. Nothing changes until you ask it to apply them. It needs Claude Code v2.1.283 or later.

## 5. Quick check (1 min)

1. When should you add a line to CLAUDE.md?
2. Why is "Format code properly" a weak instruction?
3. Can CLAUDE.md guarantee that Claude never edits a file?

<details><summary>Answers</summary>

1. When Claude repeats a mistake, a review catches something it should have known, you retype the same correction, or a new teammate would need the same fact.
2. It cannot be verified. Concrete lines such as "Use 2-space indentation" or "Run `./mvnw verify` before committing" work better.
3. No. CLAUDE.md is context, not enforcement. Use a deny permission rule or a PreToolUse hook for guarantees.

</details>

## Sources

* [How Claude remembers your project (Claude Code docs)](https://code.claude.com/docs/en/memory)
* [Claude Code overview](https://code.claude.com/docs/en/overview)
