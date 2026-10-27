---
name: standards
description: Standards workspace work through the `standards` MCP tools — answering a question from workspace data, counting or totalling records, creating or updating records, reading or filing documents attached to a record, recalling or proposing workspace memory.
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
| Memory | Pages the workspace keeps about itself, each member, each record and recurring topics. Written by the workspace; agents read and propose. |
| Role | What the connected member may read, create, update, delete, per object. Your tool list is already narrowed to it. |

**The schema is the contract.** Members add attributes, objects and select options in the app at any time, so the schema you saw yesterday is a guess today. Every attribute name, type, required field and select option comes from reading `schema:<object>` in this session.

## Tools and references

Eight tools at most: `search`, `query`, `read`, `write`, `edit_content`, `organize`, `delete`, `propose_memory`. All but `query` and `propose_memory` name things by **reference**, `<kind>:<id>`:

| Reference | Read it for |
|---|---|
| `record:<uuid>` | The record and its `version`; `include: ["memory"]` adds its memory page, `include: ["drive"]` its folders and files. |
| `document:<uuid>` | Metadata, files and extracted text. |
| `file:<uuid>` | A signed download `url`, valid 5 minutes. |
| `schema:<object>` / `schema:*` | Attributes, types, required fields, select options and views / every object you can read. |
| `memory:<name>` / `memory:*` | One memory page / the workspace profile and your own. |
| `user:<id>` | A workspace member (only with people access). |
| `guide:<kind>` | The full contract of a kind, with its pitfalls: `guide:record`, `guide:document`. |

`search` returns references as each hit's `id`, so a hit goes straight into `read`. A bare id is refused with `REFERENCE_INVALID`: prefix it.

## Orient before the first action

1. Read the server's handshake `instructions`: they list every object you can read, with its label and description. Map the user's words ("clients", "dossiers", "foyers") onto those object names.
2. `read` with `refs: ["memory:*"]`: the workspace profile and the connected member's profile — vocabulary, conventions, who is who.
3. `read` the `schema:<object>` of each object you are about to filter or write, in one call (`refs` takes up to 50).

Done when every object you will touch is named exactly as the handshake names it, and you hold its schema from this session.

## Reading

Two ways to find records, and they answer different questions:

- **Exact** — `query` with an `object` and `filters`. A complete read of the database: use it for "every deal in status Won", "contacts in 69003", anything you will list or count. Up to 1000 rows per call; paginate with `offset` until `hasMore` is false.
- **Ranked** — `search` with free text ranks by relevance across records, documents and memory (`kinds`, `objects` narrow it). Use it to find "the ACME contract" or a person by name; its hit list is partial by design, so it answers *which one*, never *how many* or *all of them*.

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

**Deletes need the user's yes.** `delete` takes one reference per call: `record:<id>` is a soft delete, `document:<id>` archives the document and its files, both restorable. Each one is announced to the user with the record's name and confirmed before the call.

## Documents and files

- `read` a `record:<id>` with `include: ["drive"]` for its folders and documents; a large drive answers `truncated` with a `next` cursor to pass back as `cursor`. `read` a `document:<id>` for its files and extracted text; `contentStatus: ocr_processing` means read again shortly.
- `read` a `file:<id>` for its download URL, valid 5 minutes: fetch it right before use and keep it out of anything durable.
- Filing goes through `organize`: `attach` (a `file:<id>` onto a `record:<id>`, optionally in a `folderPath`), `detach`, `create_folder`, `move` and `rename` (within the same drive), `init_structure` (a declared folder preset). There is no upload over MCP: `attach` takes a file already in Standards, so ask the user to upload when you only hold a local file.

## Memory

`search` with `kinds: ["memory"]` finds pages by relevance; `read` `memory:<name>` opens one. When the user states a durable fact, preference or correction ("from now on…", "always…"), call `propose_memory` with the `learning`, the user's own words as `evidenceExcerpt`, and the `record` it is about when there is one. The workspace decides where it lands; tell the user it was proposed, not saved.

## Refusals

- A tool absent from your list is outside the member's role or the connection's access level (Read, Write or Full): say which action is missing. Calling it anyway answers `Tool <name> not found`.
- `Access denied: not authorized to <action> "<object>"` is the same, for one object.
- Any other refusal is `{ error, code, expected?, next? }`: branch on `code` (`REFERENCE_INVALID`, `SCHEMA_CONFLICT`, `SCHEMA_SYSTEM_ENTITY_IMMUTABLE`…), follow `expected` or `next` when given, re-read the record on a conflict, and retry once at most. `Internal server error` with no `code` is on the server's side: say so rather than retrying in a loop.

Full reference, one file: https://docs.standards.new/llms-full.txt — the MCP tools page is https://docs.standards.new/docs/mcp/tools.
