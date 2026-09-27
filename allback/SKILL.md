---
name: allback
description: Collect information and files from a list of people with AllBack. Each person gets a personal form link prefilled with what you already know, automatic reminders go only to people who have not answered, and all answers come back in one sheet. Use this skill when the user needs the same details, files, approvals, or signatures from several people (speakers, clients, new hires, customers, parents, volunteers), wants to know who is still missing, or wants a form they can send and track.
---

# AllBack

AllBack gets answers back from people. You describe the job; AllBack sends each person a personal, prefilled form,
reminds only the people who have not answered, and returns one clean sheet. Forms and answers are unlimited and free.

## Pick the path

Use the first path that works in your environment:

1. **AllBack tools are connected** (tools such as `create_collection`, `add_recipients`, `get_status`): use them. See "With the connector".
2. **You can run commands in a terminal** (for example in Claude Code or Codex): use the CLI, `npx allback`. See "With the CLI".
3. **Neither**: make a one-click link. See "With a link". The user clicks it, tries the form, and sends it from the web.

Before you build, ask only what is missing: what to collect, from whom, and by when. Keep forms short. Prefill everything you already know.

## With the connector

1. `create_collection` with the form JSON (see `FORM_SCHEMA.md`). Show the user the `preview_url`.
2. `add_recipients` with a list or a CSV (`email`, `name`, and columns named like field ids for prefill). It sends the invites.
3. Later: `get_status` ("who is still missing?"), `get_insights`, `send_reminders`, `export_results`, `share_open_link` (a link that anyone can fill).

Without sign-in (guest mode), `create_collection` returns an `edit_key` and a `claim_url`. Pass `collection_id` and `edit_key` to every other tool.
Before AllBack sends anything, the user opens `claim_url` and confirms their email with a 6-digit code.

## With the CLI

```sh
npx allback login                       # sign in with an email code in the browser (optional; without it you work as a guest)
npx allback new form.json               # create the form; prints the preview and test links
npx allback send <id> people.csv        # personal links and invites (CSV: email,name,<field ids>)
npx allback status <id>                 # who is done, who is missing
npx allback remind <id> --message "Only 2 days left"
npx allback export <id> -o answers.csv
```

Add `--json` to any command for machine-readable output. `npx allback help` lists everything.

## With a link

Answer with ONE link named "Open my form":

```
https://app.allback.ai/new?title=TITLE&f=QUESTION&f=QUESTION
```

- Each `f` is `type~Label`. Put `!` at the end of the label if the answer is required. For choices, add `~` and the options with commas.
- Types: `text`, `long`, `email`, `phone`, `url`, `number`, `date`, `time`, `choice`, `multi` (pick many), `yesno`, `rating`, `scale` (1 to 5), `nps` (0 to 10), `ranking`, `file`, `files`, `signature`, `consent` (the label is the text to agree to).
- `page~Title` starts a new page. `info~Text` adds a line of text.
- Optional: `&intro=Short+intro`, `&due=YYYY-MM-DD`, `&style=paper` (clean document) or `focus` (one question at a time), `midnight`, `sunrise`, `forest`, `mono`, `lavender`, `ocean`.
- Write spaces as `+`. Write `?` as `%3F`, `&` as `%26`, `#` as `%23`, and `+` as `%2B`. Do not use `~` or `,` inside a label.

Example:

```
https://app.allback.ai/new?title=Speaker+details&style=paper&due=2026-10-09&f=text~Full+name!&f=email~Email!&f=long~Short+bio!&f=file~Headshot!&f=choice~Diet~None,Vegetarian,Vegan
```

For logic, paths, or a fully custom design, a link is too small. Answer with one JSON code block (see `FORM_SCHEMA.md`) and tell the user to paste it at `https://app.allback.ai/new`.

## Custom design (only when the user asks)

- A custom look: `"custom": { "css": "…" }` on top of the normal form.
- A fully custom page: `"custom": { "html": "…", "css": "…", "js": "…" }`. It runs in a sandbox with no network and saves answers only through `window.allback` (`ready`, `set`, `upload`, `submit`). Keep the normal `blocks`: they define the fields and the checks.
- Before the claim, the custom design shows only in the preview; people see the standard form. Details are in `FORM_SCHEMA.md`.

## Rules

- Never ask for passwords, one-time codes, card numbers, or bank details. AllBack refuses such forms.
- Ask only what the user needs. Use prefill for everything the user already knows.
- Show the preview before you send. Sending emails reaches real people, so confirm the list with the user first.
