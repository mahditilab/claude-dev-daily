---
layout: post
title: "Rules, permissions, hooks, mods: which one to use"
date: 2026-10-21 08:30:00 +0200
tags: [claude-code, hooks, mods, claude-md]
---

**⏱ 10 min · Level: beginner → intermediate**

## 1. The idea (2 min)

Claude Code gives you four ways to shape its behaviour. They differ in one thing: **how sure you can be that it happens.**

| Tool | What it is | Guaranteed? |
|---|---|---|
| **CLAUDE.md** | Instructions Claude reads at the start of every session | No, it's a strong suggestion |
| **Permissions** | Allow/deny rules in `settings.json` | Yes, always enforced |
| **Hooks** | Your shell command runs on fixed events (before/after a tool call, at the end of a turn…) | Yes, they always fire |
| **Mods** | Functions loaded *into* Claude Code that change what it shows, what goes into the prompt and which tool calls it makes | Yes, and they can rewrite behaviour |

Rule of thumb: **use the weakest tool that does the job.** Most things belong in CLAUDE.md. Move to permissions or hooks only when "usually" isn't good enough.

## 2. Why it matters (1 min)

Claude is non-deterministic. As a session grows, early instructions get buried. Thomas Schilling (Spring I/O 2026) said it clearly: *"Skills and CLAUDE.md can be skipped. Hooks can't."* Your guardrails (prod config, git history, build checks) should not depend on Claude remembering.

## 3. Example: a Spring Boot project (3 min)

**CLAUDE.md** for conventions:

```markdown
- Build with ./mvnw, never mvn.
- HTTP clients: RestClient, not RestTemplate.
- Tests: JUnit 5 + AssertJ.
```

**`.claude/settings.json`** with a hard rule and a hook:

```json
{
  "permissions": {
    "deny": ["Edit(**/application-prod.yml)"]
  },
  "hooks": {
    "Stop": [
      { "hooks": [{ "type": "command", "command": "./mvnw -q test || exit 2" }] }
    ]
  }
}
```

- The deny rule means Claude **cannot** edit prod config.
- The `Stop` hook runs the tests when Claude thinks it's done. Exit code `2` sends the failure back, and Claude keeps working.

In Spring terms: CLAUDE.md is the team wiki, permissions are Spring Security, hooks are AOP advice, and mods are a `BeanPostProcessor` for Claude Code itself.

## 4. Try it (3 min)

1. In a Spring project, run `claude` and type `/init` to create a CLAUDE.md.
2. Add the three lines above.
3. Add the deny rule to `.claude/settings.json`.
4. Ask Claude: *"Change the datasource URL in application-prod.yml."* Watch it get blocked and explain why.

## 5. Quick check (1 min)

1. You want Claude to *prefer* records over classes for DTOs. Which tool?
2. You must make sure Claude never runs `git push --force`. Which tool?
3. You want a status line showing whether the last Maven build failed. Which tool?

<details><summary>Answers</summary>

1. CLAUDE.md, because it's a preference.
2. A permission deny rule (or a `PreToolUse` hook), because it must be guaranteed.
3. A mod (or the `statusLine` setting), because rules and hooks can't change the UI.

</details>

## Sources

- [Claude Code docs: hooks](https://code.claude.com/docs/en/hooks)
- [Claude Code docs: settings and permissions](https://code.claude.com/docs/en/settings)
- [Mods in Claude Code (video)](https://www.youtube.com/watch?v=yUuFvL1lK4k)
- [Claude Code for Spring Developers, Spring I/O 2026 (video)](https://www.youtube.com/watch?v=PmW4-tcMQbc)
