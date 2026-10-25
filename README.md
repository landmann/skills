# Skills

Agent skills I use every day with Claude Code and Codex.

## assess-tech

Give it links to repos, APIs, products, or articles. It reads the docs and source, reads your code, and gives each one a verdict: **Implement**, **Conditional**, or **Skip**. "Worth exploring" isn't an option.

To get Implement, a tool has to pass four checks:

1. It solves a problem your code actually has.
2. It clearly beats what you do today, including the simple fix with what you already have.
3. The full cost is worth it: setup, credentials, data exposure, recurring spend, upkeep, and removal.
4. There's a complete scope to ship, not a pilot left half-built.

Most tools get Skip. That's what it's for.

For each tool the report has:

- a one-line tl;dr in the form `inputs → [black box] → outputs`
- an ASCII diagram of how it works inside
- a side-by-side with how your code does the same job today, with `file:line` references
- a ranked table of where to use it, with a verdict for each row

The first time you run it, it asks which projects to assess against and saves the list in `projects.md` in the skill folder. Say "update my projects" to change the list, or pass paths for a single run.

### Install

Claude Code:

```sh
git clone https://github.com/landmann/skills.git ~/landmann-skills
cp -r ~/landmann-skills/assess-tech ~/.claude/skills/
```

Codex: copy the same folder into `~/.codex/skills/`.

### Use

```text
/assess-tech https://github.com/some/repo https://some-api.dev/docs
```

The skill only reads. It won't install, call paid APIs, or change your code. Doing any of that takes a separate request from you.

## License

MIT
