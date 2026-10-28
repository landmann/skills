# Skills

Agent skills I use every day with Claude Code and Codex.

## assess-tech

![assess-tech sample run](examples/assess-tech-demo.gif)

Give it links to repos, APIs, products, or articles. It reads the docs and source, reads your code, and gives each one a verdict: **Implement**, **Conditional**, or **Skip**. "Worth exploring" isn't an option.

To get Implement, a tool has to pass four checks:

1. It solves a problem your code actually has.
2. It clearly beats what you do today, including the simple fix with what you already have.
3. The full cost is worth it: setup, credentials, data exposure, recurring spend, upkeep, and removal.
4. There's a complete scope to ship, not a pilot left half-built.

Most tools get Skip. That's what it's for.

A Skip doesn't end the review. It also reads the tool's source for ideas worth copying, such as a data model, a scoring rule, a UX pattern or a prompt. Each idea gets its own verdict, so you can turn down a tool and still take its best idea.

For each tool the report has:

- a one-line tl;dr in the form `inputs → [black box] → outputs`, plus how other teams use it
- an ASCII diagram of how it works inside
- a side-by-side with how your code does the same job today, with `file:line` references
- ideas worth borrowing, with ones your code already has filtered out
- one ranked table of tools and borrowed ideas, with a verdict for each row

See a [sample report on TypeSafe Jev](examples/typesafe-jev.md).

The first time you run it, it asks which projects to assess against and saves the list in `~/.config/assess-tech/projects.md`. Say "update my projects" to change the list, or pass paths for a single run.

It only reads the projects a tool can touch. A skill, prompt or other agent tooling is compared with your agent setup instead of your apps. A tool for one domain, such as social media or payments, is checked only against the projects that already have that domain.

### Install

```sh
npx skills add landmann/skills --skill assess-tech -g
```

It asks which agents to install to. Claude Code, Codex, Cursor and others are supported. Needs Node.js 22.20 or later.

### Use

```text
/assess-tech https://github.com/some/repo https://some-api.dev/docs
```

The skill only reads. It won't install, call paid APIs, or change your code. Doing any of that takes a separate request from you.

## License

MIT
