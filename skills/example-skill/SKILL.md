---
name: example-skill
description: Placeholder that lists everything a Cursor plugin skill folder can contain. Use when replacing this example with a real skill, or when someone asks what files, frontmatter, or optional directories a skill supports.
---

# Example skill

Replace this folder with a real skill. Until then, this file is the catalog of what a skill can contain.

A skill is a playbook for the agent. It does not add MCP tools. MCP lives in `mcp.json` at the plugin root. A skill can tell the agent **when** and **how** to use those tools.

## Folder

```text
skills/example-skill/
├── SKILL.md          # required — frontmatter + instructions
├── scripts/          # optional — executable helpers the agent may run
│   └── helper.sh
├── references/       # optional — extra docs loaded only when needed
│   └── REFERENCE.md
└── assets/           # optional — templates, images, sample data
    └── template.json
```

`SKILL.md` is the only required file. Folder name must match `name` in the frontmatter.

Optional siblings next to `SKILL.md` (same idea as `references/`): `reference.md`, `examples.md`. Link them from this file; do not nest references more than one level deep.

## Frontmatter

| Field | Required | What it is |
|---|---|---|
| `name` | yes | Lowercase kebab-case. Must match the folder name. Max 64 characters. |
| `description` | yes | What the skill does **and** when to use it (trigger phrases). Max 1024 characters. The agent uses this to decide relevance. |
| `paths` | no | Glob patterns. Skill is only surfaced when the agent works on matching files. |
| `disable-model-invocation` | no | `true` = slash-only (`/example-skill`). Default is auto when the chat matches `description`. |
| `icon` | no | Badge icon when used as a Custom Mode. |
| `color` | no | Badge color for Custom Mode: `default`, `green`, `cyan`, `blue`, `purple`, `magenta`, `orange`, `yellow`, `red`, `brand`. |
| `metadata` | no | Arbitrary key-value map. |

Plugin skills typically use `name` and `description` only. The other fields are for project/user skills (`.cursor/skills/`).

## Body (what to put in SKILL.md)

Write only what the agent would not already know. Typical sections:

- **When to use** — extra triggers beyond `description`
- **Safety** — which MCP server, no invented IDs, confirm before writes
- **Steps** — ordered tool calls, defaults (dates, filters), ask if ambiguous
- **Output** — exact markdown template, caps, empty-state line (see below)
- **Scripts / references** — relative paths to optional files, and whether to **run** or **read** them

## How to represent output

Put a fenced template in the skill and tell the agent to fill it, not invent a new layout. Specify title, summary line, sections, table columns, sort, cap, and the empty-state sentence.

```markdown
# <title> — <range or subject>

- Count: <n> · Other rollup: <n>

## <Section>
- <name> — <key fact>

| Col A | Col B | Col C |
|---|---:|---:|
| <value> | <value> | <value> |
```

Spell out these output rules in the skill body:

| Rule | Why |
|---|---|
| **Template** | Same headings and column order every run |
| **Labels not raw ids** | Show the names the product UI shows (`Billable`, not `2`) |
| **Units** | Hours, %, dates as ISO `YYYY-MM-DD` unless the user asked otherwise |
| **Sort** | e.g. most severe first, then alphabetical |
| **Cap** | e.g. 25 rows; then “+<n> more” |
| **Empty state** | If nothing matches: one line, do not invent rows |
| **Partial** | If a tool is missing, say so and show only what you could read |
| **Chat vs write-back** | Default is print in chat. Creating/updating records is opt-in after the user confirms |
| **Proposed writes** | List the rows that *would* be created, marked “not created”, then wait for yes |

Worked empty-state line:

```text
No conflicts in <range>.
```

Worked proposed-write block:

```markdown
## Proposed bookings (not created)
- <name> · <dates> · <hours>

Say yes to create these. Do not call write tools until then.
```

## Other things to mention in a skill

| Topic | What to write |
|---|---|
| **Defaults** | Date range, timezone, filter when the user omitted them (e.g. current week Mon–Sun) |
| **Ask first** | If several projects/people match, ask which one — do not pick silently |
| **MCP only** | Use the connected server; stop and tell the user to connect if auth fails |
| **No invented ids** | Resolve names via search; never guess numeric ids |
| **Read vs write** | Default read-only. Confirm in chat before create/update. Never delete unless asked |
| **Business math** | How to compute (e.g. copy report `%`, do not roll your own) |
| **Tie-break / buckets** | One person in only the most severe bucket; rounding rules |
| **Coded fields on write** | Send option **ids**, not labels (`billing_status: 2`) |
| **Examples** | One input → one output when format is easy to get wrong |
| **Checklist** | Numbered steps the agent can tick for a multi-step workflow |

## Optional directories

| Directory | Contains | When the agent uses it |
|---|---|---|
| `scripts/` | Bash, Python, JS, or any executable | Run a fragile/repeated step instead of generating code |
| `references/` | Extra markdown/docs | Load on demand so `SKILL.md` stays short |
| `assets/` | Templates, images, JSON fixtures | Copy or fill as a starting file |

Keep `SKILL.md` under ~500 lines. Put long API notes in `references/`.

## What a skill can do

- Match a user request from `description` or `/skill-name`
- Tell the agent which MCP tools to call, in what order, with which filters
- Constrain writes (confirm first, never delete)
- Force an output format
- Point at bundled scripts, references, and assets
- Stay on for a session as a Custom Mode

## What a skill cannot do

- Expose new MCP tools or authenticate — that is `mcp.json` / OAuth
- Query ERS (or any product) if MCP is disconnected
- Guarantee the model follows it; keep instructions short and imperative

## Replace this example

1. Delete `skills/example-skill/`.
2. Add `skills/<name>/SKILL.md` with `name` + `description`.
3. Add `scripts/`, `references/`, or `assets/` only if the playbook needs them.
