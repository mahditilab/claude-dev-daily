---
layout: post
title: "Path-scoped rules in .claude/rules/: the right rule at the right time"
date: 2026-10-26 08:30:00 +0100
tags: [claude-code, rules, memory, java]
---

**⏱ 10 min · Level: intermediate**

## 1. The idea (2 min)

Last time we said: keep `CLAUDE.md` short, and put rules that matter for only one part of the codebase somewhere else. This is that "somewhere else".

Put markdown files in `.claude/rules/` in your project. Each file covers one topic, for example `testing.md` or `api-design.md`. Claude Code finds all `.md` files there, also in subfolders.

There are two kinds of rule files:

* **No `paths` frontmatter:** loaded at launch, with the same priority as `.claude/CLAUDE.md`. This is just CLAUDE.md split into topics.
* **With `paths` frontmatter:** loaded only when Claude works with a matching file. The docs say a path-scoped rule loads when Claude uses Read, Write or Edit on a matching file (and also for simple single-file reads such as `cat` or `head`).

```markdown
---
paths:
  - "src/main/java/**/web/**/*.java"
---

# Web layer rules
- ...
```

Good to know from the docs:

* `paths` is the **only** frontmatter field Claude Code reads in a rule. Other fields are ignored.
* `paths` takes a YAML list or a comma-separated string. Brace expansion works: `src/**/*.{java,kt}`.
* Patterns are relative to the folder that contains the `.claude` folder.
* If the YAML is broken, the rule loads as if it had no `paths`. Run `claude --debug` to see the error.
* Personal rules can live in `~/.claude/rules/` and apply to all your projects.

## 2. Why it matters for developers (1 min)

A Spring Boot service has areas with different rules. Controllers need validation and a standard error format. Repositories need query rules. Tests need Testcontainers. Flyway migrations have their own naming rules.

If all of that sits in CLAUDE.md, it costs context in every session, even when you only fix a typo in a README. Path-scoped rules keep the always-on file small and still give Claude the right rules at the moment it edits a controller, a test or a migration.

Rules also help your team: one file per topic is easier to review in a pull request than one long file.

## 3. Example (3 min)

Layout for an orders service that uses package-by-feature (`com.example.orders.checkout`, `com.example.orders.catalog`, ...):

```text
orders-service/
├── CLAUDE.md                     # 5-10 lines, always loaded
└── .claude/
    └── rules/
        ├── web.md                # only for controllers
        ├── testing.md            # only for tests
        └── migrations.md         # only for Flyway files
```

`.claude/rules/web.md`:

```markdown
---
paths:
  - "src/main/java/**/*Controller.java"
---

# Controller rules
- Validate request bodies with `@Valid` and Jakarta Bean Validation.
- Return DTO records, never JPA entities.
- Errors go through the global `@RestControllerAdvice`. No try/catch in controllers.
```

`.claude/rules/testing.md`:

```markdown
---
paths:
  - "src/test/java/**/*.java"
---

# Test rules
- Integration tests extend `AbstractIntegrationTest` (Testcontainers PostgreSQL). Never H2.
- Name tests `shouldDoSomething_whenCondition`.
- Single test: `./mvnw -Dtest=OrderServiceTest test`.
```

`.claude/rules/migrations.md`:

```markdown
---
paths:
  - "src/main/resources/db/migration/**/*.sql"
---

# Flyway rules
- Never edit a migration that is already merged. Add a new one.
- File name: `V<next number>__<short_description>.sql`.
- Make changes backward compatible: add a column first, drop it in a later release.
```

Notice how this works with package-by-feature: because the patterns use `**`, the rules apply to every feature package without you listing each one.

Now, when you ask Claude to "add a `DELETE /orders/{id}` endpoint", it reads or edits a `*Controller.java` file, and `web.md` is loaded into the context. The migration rules stay out until Claude touches the `db/migration` folder.

One more tip from the docs: if something is a multi-step task such as "how to cut a release", make it a **skill** instead. Rules load into context every session or when matching files are opened. Skills load only when invoked or when Claude finds them relevant.

## 4. Try it (3 min)

1. Open a Spring Boot project and create `.claude/rules/testing.md` with the `paths` frontmatter above and two or three real rules from your project.
2. Start Claude Code in the project. Run `/context` and look at the list of memory files. A path-scoped rule should not be loaded yet, because no matching file was opened.
3. Ask: "Read `src/test/java/.../SomeTest.java` and tell me which testing rules you follow." Claude should now know your rule.
4. Add a second rule file without `paths` (for example `logging.md` with one line about SLF4J). Restart and check that this one is loaded at launch.
5. If a rule never seems to apply, check that the glob is relative to the project root, and run `claude --debug` to see YAML errors.

## 5. Quick check (1 min)

1. What is the difference between a rule file with `paths` and one without?
2. Which frontmatter field does Claude Code read in a rule file?
3. A "how to release" procedure with many steps: rule or skill?

<details><summary>Answers</summary>

1. Without `paths` the rule loads at launch like CLAUDE.md. With `paths` it loads only when Claude works with a matching file.
2. Only `paths`. Other fields are ignored.
3. A skill. Skills load only when invoked or relevant, and fit multi-step procedures better than rules.

</details>

## Sources

* [Memory, CLAUDE.md and rules (official docs)](https://code.claude.com/docs/en/memory)
* [Claude Code docs overview](https://code.claude.com/docs/en/overview)
