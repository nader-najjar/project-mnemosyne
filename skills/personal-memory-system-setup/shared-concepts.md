# Shared Concepts

This system turns AI conversations into durable, source-backed memory. Setup configures sources, standing instructions, schedules, and optional backups. Discovery saves conversation snapshots and queues them; backfill supplies older conversations; memorization processes snapshots into topic pages; recall loads those pages and checks system health. Captures preserve the evidence. A shared lock coordinates writers, Git preserves committed state, troubleshooting recovers abandoned runs, and backups preserve the repository outside the memory folder.

## Scratch

Use a unique per-run directory under the OS temporary directory for ad hoc temporary files and working notes. Everything there must be disposable and reconstructible; never store required state there.

Each run MUST remove its directory when it finishes, including on failure. Do not delete another run's files.

## Source Staging

`<memory-folder>/source-staging/<source-type>/` is where that source’s export or sync job puts conversation files for discover to read.

## Memory Configuration

The memory folder’s configuration is `<memory-folder>/config.json`:

```json
{
  "sources": [
    {
      "identifier": "<source-type-identifier>",
      "location": "<memory-folder>/source-staging/<source-type>/",
      "staged_through": null
    }
  ],
  "backup_file": null,
  "google_drive_publishing": false
}
```

`staged_through` is the last successful staging run’s start time as an ISO 8601 UTC timestamp, or `null` before the first successful run.

`backup_file` is the absolute path to `<backup-folder>/<memory-folder-name>.bundle`, where `<memory-folder-name>` is the memory folder’s final path component. Use `null` when the owner declines backup.

`google_drive_publishing` is `true` when Google Drive publishing is enabled, or `false` when disabled. The Google Drive copy of the memory is used to allow non-local AI tools to utilize the memory.

## Source Type Definitions

Source type definitions are the Markdown files in `<memory-folder>/operations/shared/source-types/`. Match each configured source's `<source-type-identifier>` to a definition's **Identifier** section to locate its definition.

## Queue

The queue is `<memory-folder>/operations/queue.json`: a UTF-8 JSON object:

```json
{
  "discovered_through": "<UTC-timestamp>",
  "items": [
    {"path": "<relative-capture-path>"}
  ]
}
```

`discovered_through` is the timestamp through which the last successful discovery scan searched, inclusive, recorded as an ISO 8601 UTC timestamp.

Each item's `path` is relative to the memory folder and appears at most once. Items are ordered by when they were added, oldest first. An empty `items` array means no pending ingestion.

## Backfill

The backfill worklist is `<memory-folder>/operations/backfill.json`: a UTF-8 JSON object:

```json
{
  "items": [
    {"path": "<relative-capture-path>"}
  ]
}
```

Items follow the same format as the queue and identify saved snapshots awaiting admission to it. Paths are relative to the memory folder and appear at most once. Items are processed in list order. An empty `items` array means no pending backfill.

## Capture Naming

The filename is `<source-type>--<UID>--<source-modified-time>--<sha256>.jsonl`, where `<source-type>` comes from the source definition’s **Identifier** section, `<UID>` is dynamically specified from upstream at time of capture creation, `<source-modified-time>` is the source file’s filesystem modification time observed when selecting the item, expressed in UTC in `YYYYMMDDTHHmmssffffffZ` format, and `<sha256>` is the full lowercase hexadecimal SHA-256 hash of the capture’s complete uncompressed bytes. For the same conversation, equal hashes identify unchanged content; modification time does not determine whether the content changed.

## Memory Pages

Memory pages are Markdown files under `<memory-folder>/memory/`, organized by topic. Each page has YAML front matter with:

- `title`: the topic’s primary name.
- `aliases` (optional): a list of alternative names, abbreviations, or former names that should find this same topic during recall.
- `parent` (optional): the path of the memory page whose topic this page is a facet of, relative to `memory/`.

A facet covers one aspect of a broader topic, such as a project’s deployment. It has one parent page; a parent may have many facets. Standalone pages omit `parent`. The `parent` field defines the hierarchy independently of folder structure. Other relationships use links in the body.

The body is freeform Markdown with descriptive headings.

Citations are relative links to captured conversations, with record identifiers or line numbers as anchors.

`## Superseded` holds previous information, its dates and sources, and what replaced it.

## Commit Changes

Run `git add .` from `<memory-folder>` and commit with a message describing the operation. Skip the commit if nothing changed.

## Lock Owner

Each acquired lock contains only a regular, non-symlink `.lock/owner.json`:

```json
{"token": "<new UUID>", "tool": "<tool name>", "session_id": "<current session ID>"}
```

Record your current session ID, using judgment based on the tool.

## Acquire Lock

1. Prepare a fresh `Lock Owner` record. If the current session ID is unavailable, report it and return `false`.
2. Acquire the lock with `mkdir "<memory-folder>/.lock"`. If it fails, return `false`. Retain the directory's identity and your token.
3. Create `owner.json` exclusively inside it and read it back to verify the record. On failure, remove only your newly created owner file, then `rmdir` the same directory and return `false`. Report cleanup failures without deleting unexpected contents.
4. Run `git -C "<memory-folder>" status --porcelain=v1 --untracked-files=all`. If it fails or returns any output, release the lock and return `false`.
5. Return `true`.

## Release Lock

1. Verify the same directory contains only a regular, non-symlink `owner.json` with your saved token. If anything differs or is uncertain, leave the lock unchanged and report it.
2. Remove only that verified `owner.json`, then run `rmdir "<memory-folder>/.lock"`. Never recursively delete the lock or remove unexpected contents.
