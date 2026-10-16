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

**The schema is the contract.** Members add attributes, objects and select options in the app at any time, so the schema you saw yesterday is a guess today. Every attribute name, type, required field and select option comes from `get_schema` in this session.

## Orient before the first action

1. Read the server's handshake `instructions`: they list every object you can read, with its label and description. Map the user's words ("clients", "dossiers", "foyers") onto those object names.
2. Call `get_memory` with no argument: it returns the workspace profile and the connected member's profile — vocabulary, conventions, who is who.
3. Call `get_schema` on each object you are about to filter or write.

Done when every object you will touch is named exactly as the handshake names it, and you hold its schema from this session.

## Reading

Two ways to find records, and they answer different questions:

- **Exact** — `search_records` with `filters` and no `query`. A complete read of the database: use it for "every deal in status Won", "contacts in 69003", anything you will list or count. Paginate with `offset` until `hasMore` is false.
- **Ranked** — `query` (in `search_records` or `global_search`) ranks by text relevance. Use it to find "the ACME contract" or a person by name; its hit list is partial by design, so it answers *which one*, never *how many* or *all of them*.

Then:

- Known id → `get_record`; it also returns the record's memory page when one exists.
- Several objects → one `search_records` per object, in parallel; `global_search` when you do not know the object.
- Ask for `fields` you need; a full record can be large.

## Counting and totals

`aggregate_records` answers counts, sums, averages, min and max in one exact call, grouped by a status, select, user, relation or date bucket (`day` … `year`). Reach for it whenever the user asks *how many*, *how much*, or *by month / by owner* — paging records and adding them up is slower and silently wrong past a page. For a bare count, `search_records` with `filters` and `count: true` also returns `{ total, totalIsExact: true }`.

## Writing

- One record → `create_record` / `update_record` (only the fields in `data` change). Many → `bulk_create_records` / `bulk_update_records`, up to 100 per call; refused ids come back in `errors` while the rest are written, so report both.
- Relations take record ids: resolve each one with an exact search first.
- Select and status values must be one of the options `get_schema` returned.
- Richtext (notes, descriptions) → `edit_content`: `replace` a passage that appears exactly once, or `insert` after one. Only the change travels, and a concurrent edit is refused with `SCHEMA_CONFLICT` rather than overwritten.
- Read back what you wrote when the user will act on it.

**Deletes need the user's yes.** `delete_record` and `archive_document` are soft and restorable, yet each one is announced to the user with the record's name and confirmed before the call. Bulk delete does not exist over MCP.

## Documents and files

- `list_record_drive` shows a record's folders and documents with their ids; `read_document` returns a document's files and extracted text. `contentStatus: ocr_processing` means call again shortly.
- `get_file_url` gives one file's download URL, valid 5 minutes: fetch it right before use and keep it out of anything durable.
- Filing: `create_folder`, `update_document` (rename or move within the same drive), `attach_file`, `remove_file`. `attach_file` takes a `fileId` from a REST or CLI upload; uploading bytes happens outside MCP, so ask the user to upload when you only hold a local file.

## Memory

`search_memories` finds pages by relevance; `get_memory` opens one by name. When the user states a durable fact, preference or correction ("from now on…", "always…"), call `propose_memory` with the learning and the user's own words as evidence. The workspace decides where it lands; tell the user it was proposed, not saved.

## Refusals

- A tool absent from your list is outside the member's role: say which action the role lacks.
- `Access denied: not authorized to <action> "<object>"` is the same, for one object.
- Any other error carries a `code` (`SCHEMA_CONFLICT`, `SCHEMA_SYSTEM_ENTITY_IMMUTABLE`, `DOCUMENT_FILE_NOT_FOUND`…): branch on the code, re-read the record on a conflict, and retry once at most.

Full reference, one file: https://docs.standards.new/llms-full.txt — the MCP tools page is https://docs.standards.new/docs/mcp/tools.
