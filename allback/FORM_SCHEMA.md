# AllBack form schema

The agent sends this JSON to create a form. The MCP tool `create_collection` (next step) takes it as `form`.
Source of truth: `src/shared/schema.ts`. The server rejects a form with a clear error when a reference or value is wrong.

## Minimal form

```json
{
  "title": "Speaker details",
  "blocks": [
    { "id": "full_name", "type": "short_text", "label": "Full name", "required": true },
    { "id": "headshot", "type": "file", "label": "Headshot", "accept": ["image/*"] }
  ]
}
```

`blocks` is a shortcut for one page. Use `pages` for steps and paths.

## Style: "make it like …"

| The user says | Set |
|---|---|
| "like Tally", "simple", "like a doc" | `theme.preset: "paper"` (layout `classic`) |
| "like Typeform", "one question at a time", "conversational" | `theme.preset: "focus"` (layout `conversational`) |
| "dark", "sleek" | `midnight` |
| "warm", "friendly" | `sunrise` (conversational, serif) |
| "calm", "green", "nature" | `forest` |
| "brutalist", "techy", "monospace" | `mono` |
| "playful", "soft", "purple" | `lavender` (conversational, centered) |
| "corporate", "blue", "trustworthy" | `ocean` |

Override any token after the preset: `background`, `surface`, `text`, `muted`, `accent`, `accentText`, `border`, `error` (hex colors), `font` and `headingFont` (`system`, `inter`, `grotesk`, `serif`, `rounded`, `mono`), `radius` (`none` … `full`), `surfaceStyle` (`card`, `plain`), `align` (`left`, `center`), and `logo`, `cover`, `backgroundImage` (https URLs, or paths on our site like `/demo/venue-cabin.svg`).
Set `layout` to force `classic` or `conversational` with any preset.

## Field types

| Type | Answer value | Options |
|---|---|---|
| `short_text`, `long_text` | string | `minLength`, `maxLength`, `placeholder` |
| `email`, `phone`, `url` | string | validated format |
| `number` | number | `min`, `max`, `step`, `unit` |
| `choice` | option id, or list of ids | `options` (strings or `{id, label, image}`), `multiple`, `minSelect`, `maxSelect`, `allowOther`, `display` (`auto`, `list`, `dropdown`, `cards`), `shuffle` |
| `yes_no` | boolean | `yesLabel`, `noLabel` |
| `rating` | 1…max | `max` (3–10), `icon` (`star`, `heart`, `thumb`, `bolt`) |
| `scale` | min…max | `min` (0 or 1), `max` (≤10), `minLabel`, `maxLabel`. NPS = 0–10. |
| `ranking` | list of option ids | `options` |
| `matrix` | `{rowId: columnId}` | `rows`, `columns` |
| `date`, `time` | `YYYY-MM-DD`, `HH:MM` | `min`, `max` (date) |
| `file` | list of file ids | `multiple`, `maxFiles`, `accept` (`image/*`, `application/pdf`, …), `maxSizeMb` (≤25) |
| `signature` | list with one file id | — |
| `consent` | `true` | `text` (markdown links allowed) |
| `hidden` | string | value comes only from prefill; never shown |

All fields take `id` (lowercase, `_`), `label`, `description`, `required`, `showIf`.
Content blocks: `heading`, `text` (markdown: `**bold**`, `*italic*`, `[link](https://…)`), `divider`, `image` (`src`), `document`.

### Document (read and confirm)

```json
{ "type": "document", "id": "doc_policy", "title": "Travel policy 2026", "body": "# Booking\n- Book 14 days ahead", "required": true }
{ "type": "document", "id": "doc_handbook", "title": "Handbook", "src": "https://example.com/handbook.pdf", "required": true }
```

- Give `body` (markdown with `#` headings, `-` and `1.` lists, paragraphs) or `src` (an https link or a same-site path). A PDF link shows inside the form, with an "Open in a new tab" link.
- `id` is optional. Without it, the id comes from the title (`doc_travel_policy`).
- `required: true`: a text document counts as read when the person scrolls to the end. A linked document counts when it stayed on screen for 3 seconds or the person opened the link. The server checks this again on submit.
- Pattern: a required document, then a required `consent`, then an optional `signature`. `get_status` lists who read each document.

## Logic

**Show a field only when …** — `showIf` on any block:

```json
{ "id": "workshop_size", "type": "number", "label": "Seats", "showIf": { "field": "format", "op": "eq", "value": "Workshop" } }
```

**Paths** — `next` on a page. The first matching rule wins. No match goes to the next page.

```json
{ "id": "travel", "blocks": [...], "next": [{ "if": { "field": "needs_travel", "op": "eq", "value": false }, "to": "extras" }] }
```

`to` is a page id, `"end"`, or `"end:<ending id>"`.

**Conditions** nest: `{ "all": [...] }`, `{ "any": [...] }`, `{ "not": {...} }`.
Operators: `eq`, `neq`, `in`, `not_in`, `contains`, `not_contains`, `gt`, `gte`, `lt`, `lte`, `empty`, `not_empty`.
Choice values match by option id or label, without case. Use `recipient.name`, `recipient.first_name`, `recipient.email` as fields too.

**Endings** — a jump like `"end:regrets"` picks that ending. Otherwise: the first ending whose `showIf` matches, else the first ending without `showIf` that no jump points to:

```json
"endings": [
  { "id": "fan", "title": "You made our day!", "showIf": { "field": "nps", "op": "gte", "value": 9 } },
  { "id": "default", "title": "Thanks for your feedback." }
]
```

## Piping

Use `{{field_id}}` or `{{recipient.first_name}}` in titles, labels, descriptions, and endings: `"Thanks {{full_name}}!"`.

## Conversational extras

- `welcome: { "text": "...", "button": "Start" }` sets the first screen.
- A page with `"group": true` shows all its fields on one screen, not one per screen. Use it for related short fields (name + email).

## Settings

`settings.submitLabel` (default "Submit"), `settings.showProgress` (default true), `settings.highlightPrefill` (default true).

## Custom design (optional)

Only when the user asks for a fully custom look. Guests can build it, but before the claim only the preview shows it: people see the standard form until someone confirms their email.

**Level 1, custom CSS** on top of the normal form:

```json
"custom": { "css": ".ab-btn { border-radius: 0; background: #ff0080; }" }
```

Useful classes: `.ab-shell`, `.ab-main`, `.ab-sheet`, `.ab-h1`, `.ab-lead`, `.ab-q`, `.ab-input`, `.ab-btn`, `.ab-choice-row`, `.ab-error`.
`url()` works only with `data:image/…`. `@import` is not allowed. The CSS can load nothing from outside.

**Level 3, a fully custom page** in a sandbox:

```json
"custom": { "html": "<input id=nm><button id=go>Send</button>", "css": "…", "js": "go.onclick = async () => { await allback.set('full_name', nm.value); await allback.submit(); }" }
```

- Keep the normal `blocks`. They define the fields, the checks, the export, and the "Standard view" that people can always open.
- The page runs in a sandbox with no network: no `fetch`, no outside scripts or images, no links away, no cookies or storage.
- The only way to save answers is `window.allback`:
  - `await allback.ready()`: `{ form, fields, answers, recipient, dueAt, preview }`, with the prefill in `answers`.
  - `await allback.set(fieldId, value)`: `{ ok, error }`. Choice answers can be the option id or its label.
  - `await allback.upload(fieldId, file)`, `await allback.documentRead(docId)`, `allback.standardView()`.
  - `await allback.submit()`: `{ ok, errors, ending }`. The same server checks as for every form.
- Images: use `data:` URIs.
- Pages that ask for passwords, codes, card numbers, or bank details are refused.
- Our bar above the page always shows the confirmed sender, "Standard view", and "Report".

