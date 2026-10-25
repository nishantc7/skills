# skills

Agent skills for Claude Code and other coding agents.

## reviewing-pr-decisions

Lists the judgment calls a pull request made, as before/after code pairs with
one question each, and renders them as an HTML page. It never says whether to
merge. A second agent hunts for security and performance defects in the same
diff. See `reviewing-pr-decisions/example-review.html` for a sample page.

Why I built it: [Code review after coding agents](https://nishantc7.github.io/writing/2026/10/26/code-review-after-coding-agents/).

Needs the [GitHub CLI](https://cli.github.com/) (`gh`), logged in.

## Install

Copy a skill folder into `~/.claude/skills/` (all projects) or
`.claude/skills/` (one project):

```bash
git clone https://github.com/nishantc7/skills.git
cp -r skills/reviewing-pr-decisions ~/.claude/skills/
```

Then ask your agent to "review PR #123".
