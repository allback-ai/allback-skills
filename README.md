# AllBack skill

An agent skill for [AllBack](https://allback.ai): ask once, get it all back.
It teaches Claude and other agents to collect information and files from a list of people: a personal, prefilled form
for each person, automatic reminders only to the people who have not answered, and one clean sheet at the end.

## Install

- **Claude (app or web):** download [allback-skill.zip](https://allback.ai/allback-skill.zip), then Settings > Capabilities > Skills > Upload.
- **Claude Code:** copy the `allback` folder into `~/.claude/skills/` (for you) or `.claude/skills/` (for one project).
- **Other agents:** point them to `allback/SKILL.md`. It is plain Markdown.

## What the agent learns

- When to use AllBack, and which path to take: the AllBack connector, the CLI ([`npx allback`](https://github.com/allback-ai/allback-cli)), or a one-click link.
- The one-click link format and the JSON form format (`allback/FORM_SCHEMA.md`), with logic, paths, and optional custom design.
- The rules: short forms, prefill what you know, and never ask for passwords, codes, or card or bank details.

## Examples

`examples/` has ready forms: a fully custom team picker, custom CSS, read and confirm, and an RSVP with logic.

## License

MIT
