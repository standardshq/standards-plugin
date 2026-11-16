---
name: standards
description: Standards workspace work through the `standards` MCP tools — answering a question from workspace data, counting or totalling records, creating or updating records, reading or filing documents attached to a record, recalling or keeping workspace memory, running a routine or an agent, drafting a change to the workspace's structure.
---

# Standards

## What Standards is

Standards ([standards.new](https://standards.new)) is a business workspace — CRM, documents, drives, email, forms and AI agents in one app — whose data model is declared, not fixed. Each team shapes its own: a sales team's workspace holds `contacts`, `companies` and `deals`, an operations team's holds `incidents` and `suppliers`. The `standards` MCP server opens exactly one workspace, with the rights of the member who connected.

| Word | Meaning |
|---|---|
| Workspace | One tenant: its schema, records, members, memory. Your connection is bound to one. |
| Object | A type of record, kebab-case and usually plural (`contacts`, `deals`). Declared in code or created by a member in the app, so the list differs per workspace. |
| Attribute | A typed field on an object: text, richtext, number, checkbox, date, phone, currency, status, select, multiselect, location, user, relation, and a few more. Validated on every write. |
| Record | One row of an object: a UUID, timestamps, a `version`, and its attributes. |
| Document | A titled pack of files, always attached to one record, filed in that record's drive (folders + documents). |
| Memory | Pages the workspace keeps about itself, each member, each record and recurring topics, one dated line per fact. Agents read them and add lines. |
| Role | What the connected member may read, create, update, delete, per object. Your tool list is already narrowed to it. |

**The schema is the contract.** Members add attributes, objects and select options in the app at any time, so the schema you saw yesterday is a guess today. Every attribute name, type, required field and select option comes from reading `schema:<object>` in this session.

## Tools and references

Seven tools work on data: `search`, `query`, `read`, `write`, `edit_content`, `organize`, `delete`. `start` runs a routine or an agent, and four more draft the workspace's structure (see "Routines and agents" and "Changing the workspace's structure"). Your list holds only those the member's role allows. All but `query` name things by **reference**, `<kind>:<id>`:

| Reference | Read it for |
|---|---|
| `record:<uuid>` | The record and its `version`; `include: ["memory"]` adds its memory page, `include: ["drive"]` its folders and files. |
| `document:<uuid>` | Metadata, files and extracted text. |
| `file:<uuid>` | A signed download `url`, valid 5 minutes. |
| `schema:<object>` / `schema:*` | Attributes, types, required fields, select options and views / every object you can read. |
| `memory:<name>` / `memory:*` | One memory page / the workspace profile and your own. |
| `user:<id>` | A workspace member (only with people access). |
| `agent:<id>` / `routine:<id>` | An agent's or a routine's setup; `include: ["runs"]` lists its runs. |
| `guide:<kind>` | The full contract of a kind, with its pitfalls: `guide:record`, `guide:document`, `guide:schema`. |

`search` returns references as each hit's `id`, so a hit goes straight into `read`. A bare id is refused with `REFERENCE_INVALID`: prefix it.

## Orient before the first action

1. Read the server's handshake `instructions`: they list every object you can read, with its label and description. Map the user's words ("clients", "dossiers", "foyers") onto those object names.
2. `read` with `refs: ["memory:*"]`: the workspace profile and the connected member's profile — vocabulary, conventions, who is who.
3. `read` the `schema:<object>` of each object you are about to filter or write, in one call (`refs` takes up to 50).

Done when every object you will touch is named exactly as the handshake names it, and you hold its schema from this session.

## Reading

Two ways to find records, and they answer different questions:

- **Exact** — `query` with an `object` and `filters`. A complete read of the database: use it for "every deal in status Won", "contacts in 69003", anything you will list or count. Up to 1000 rows per call; paginate with `offset` until `hasMore` is false.
- **Ranked** — `search` with free text ranks by relevance across records, memory and members (agents and routines only when `kinds` names them; `kinds`, `objects` narrow it). Use it to find "the ACME deal" or a person by name; its hit list is partial by design, so it answers *which one*, never *how many* or *all of them*. Snippets are excerpts: `read` a hit before answering from its content.

Then:

- Known references → one `read` with all of them; members fail independently, each with its own `code`.
- Ask for the `fields` you need; a full record can be large.

## Counting and totals

`query` with `aggregate` answers counts, sums, averages, min and max in one exact call: `aggregate: { measure: "count" }`, or `sum` / `avg` / `min` / `max` over a number or currency `attribute`, with `filters`. `groupBy` takes a status, select, user, relation or date attribute, or `{ attribute, bucket }` with `day` … `year`. Reach for it whenever the user asks *how many*, *how much*, or *by month / by owner* — paging records and adding them up is slower and silently wrong past a page. Amounts in several currencies come back per currency in `byCurrency`; never add them together.

## Writing

- `write` with `action: "create"`, an `object` and `items: [{ data }]`; or `action: "update"` with `items: [{ ref, data }]`, or `refs` plus one `data` to give several records the same values. Only the fields in `data` change. Up to 100 records per call; refused members come back in `errors` while the rest are written, so report both.
- Relations take record ids: resolve each one with an exact `query` first.
- Select and status values must be one of the options the schema listed.
- Richtext (notes, descriptions) → `edit_content` with `target` `record:<id>` and the `attribute`: `replace` a passage that appears exactly once, or `insert` after one. Only the change travels, and a concurrent edit of the same passage is refused with `SCHEMA_CONFLICT` rather than overwritten.
- Read back what you wrote when the user will act on it.

**Deletes need the user's yes.** `delete` takes one reference per call (no bulk delete over MCP), so a list of deletions is one call each: `record:<id>` is a soft delete, `document:<id>` archives the document and its files, both restorable. Announce what will go, by name, and get the user's confirmation of that exact list before the call.

## Documents and files

- `read` a `record:<id>` with `include: ["drive"]` for its folders and documents; a large drive answers `truncated` with a `next` cursor to pass back as `cursor`. `read` a `document:<id>` for its files and extracted text; `contentStatus: ocr_processing` means read again shortly.
- `read` a `file:<id>` for its download URL, valid 5 minutes: fetch it right before use and keep it out of anything durable.
- Filing goes through `organize`: `attach` (a file onto a `record:<id>`, optionally in a `folderPath`), `detach`, `create_folder`, `move` and `rename` (within the same drive), `init_structure` (a declared folder preset).
- `attach` takes a file three ways: `fileId: "file:<id>"` for one already in Standards; `file`, a `data:<type>;name=<file name>;base64,<bytes>` URI, up to 512 KiB; or, larger, `upload: { fileName, mimeType, size }`, which answers an HTTPS URL to `PUT` the bytes to and the `upload:<id>` to attach with afterwards.
- `organize` with `file_under` files a record under another record of the same object (`parent: null` moves it back to the top level); the record and its sub-records take the parent's access.

## Memory

`search` with `kinds: ["memory"]` finds pages by relevance; `read` `memory:<name>` opens one, and a `read` of up to 5 records brings each one's memory page along. When the user states a durable fact, preference or correction ("from now on…", "always…"), `write` with `action: "add"`, `ref` `memory:user:self` (the member) or `memory:record:<id>` (what it is about), and `data: { text, type, source }`: one sentence, a type among `préférence`, `relation`, `contexte` or `décision` (a `consigne` is refused over MCP), and the `source` reference it came from, which is required. The answer's `landedOn` names the page the line reached: name it when you tell the user. Over MCP lines are only added: `rewrite` and `remove` belong to the workspace's memory routines, or are done in the app.

## Routines and agents

`search` with `kinds: ["routine"]` or `["agent"]` and no query lists them; `read` one for its setup, with `include: ["runs"]` for its runs. `start` with `target` `routine:<id>` or `agent:<id>`, and a required `text` as its first message, runs it now, as its Run button does. Nothing reports back: read its runs to follow it.

## Changing the workspace's structure

The workspace's structure changes through a **draft** the member watches and publishes themselves; nothing reaches the workspace before that. `get_architect_draft` returns the draft and the `guard` to write against; `draft_schema` writes a change set, after a `read` of `guide:schema`; `search_icons` returns valid icon names, never guess one. Over MCP a draft takes `+`, `-` and `~` blocks over objects, attributes, sample records, records and bundles: view and page blocks are refused, and those change from the app. `discard_architect_draft` throws the whole draft away, the member's own edits included: only when they ask for exactly that.

## Refusals

- A tool absent from your list is outside the member's role or the connection's access level (Read, Write or Full): say which action is missing. Calling it anyway answers `Tool <name> not found`.
- `Access denied: not authorized to <action> "<object>"` is the same, for one object.
- Any other refusal is `{ error, code, expected?, next? }`: branch on `code` (`REFERENCE_INVALID`, `SCHEMA_CONFLICT`, `SCHEMA_SYSTEM_ENTITY_IMMUTABLE`…), follow `expected` or `next` when given, re-read the record on a conflict, and retry once at most. `Internal server error` with no `code` is on the server's side: say so rather than retrying in a loop.

Full reference, one file: https://docs.standards.new/llms-full.txt — the MCP tools page is https://docs.standards.new/docs/mcp/tools.
