---
name: standards
description: Use when working with a standards.new workspace through the `standards` MCP server — finding, reading, creating or updating records (contacts, companies, deals, tasks, any custom object), reading or editing documents and files attached to records, or answering a question from workspace data.
---

# Working in a standards.new workspace

The `standards` MCP server (`https://api.standards.new/v1/mcp`) exposes one workspace: its objects, their records, the documents and files attached to them, and the workspace memory. The handshake `instructions` list the objects you may read; trust that list over assumptions.

## Before you act

1. Call `get_memory` with no argument: it returns your profile and the workspace profile (conventions, vocabulary, who is who).
2. Object names are kebab-case and usually plural (`contacts`, `companies`, `tasks`). Call `get_schema` on an object before writing to it: attribute names, types, required fields and select options come from there, never from a guess.

## Finding records

- Known id → `get_record`. Do not search for a record you can fetch.
- Filtered list on one object → `search_records` with the schema's attribute names.
- Free text across objects → `global_search`.
- Ask for the fields you need; a record set is paginated, so say when you stopped at a page.

## Writing

- One record → `create_record` / `update_record`; many → `bulk_create_records` / `bulk_update_records`.
- Relation attributes take record ids: resolve them with a search first.
- `delete_record` moves the record to the trash (reversible), but **confirm with the user before every delete**, and before any bulk update that overwrites values.

## Documents and files

- `list_record_drive` shows a record's folders and documents; `read_document` returns a document's content and its file ids.
- `get_file_url` turns one file id into a short-lived download URL. Never paste that URL somewhere durable.
- `update_document` / `edit_content` change content; `attach_file` / `remove_file` manage files on a document.

## Memory

Use `search_memories` to recall workspace conventions, and `propose_memory` when the user states a durable fact or preference worth keeping. A proposal is reviewed; it is not stored silently.

## When something fails

Errors share one envelope with a `code`. `FORBIDDEN` means the connected member's role lacks that grant — say so rather than retrying. Full reference: https://docs.standards.new/llms-full.txt
