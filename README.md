# Claude Dev Daily

A 10-minute lesson every weekday on what's new in Claude and Claude Code, written for developers (Java/Spring examples).

Site: https://mahditilab.github.io/claude-dev-daily/

## How it works

- A scheduled Claude task runs Mon–Fri at 08:30 (Europe/Berlin) on the owner's Mac.
- It checks for news (Claude Code changelog, the Claude YouTube channel, Anthropic docs). If something developer-relevant shipped since the last lesson, the lesson covers it; otherwise it takes the next open topic from [`curriculum.md`](curriculum.md).
- It writes `_posts/YYYY-MM-DD-<slug>.md`, ticks the topic in `curriculum.md` and pushes to `main`. GitHub Pages builds the site.

## Add a topic

Add a line `- [ ] Your topic` to `curriculum.md`. Topics are taken top to bottom.
