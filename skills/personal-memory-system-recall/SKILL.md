---
name: personal-memory-system-recall
description: Load remembered context about requested topics into the current chat, read-only.
---
# Personal Memory System Recall

Inputs:
  1. Topics to recall

Definitions:
  1. Local Memory: `<local-path>`

This skill is read-only. Do not change local memory or Drive files. Temporary downloads/extraction are disposable; remove only this run’s files when recall ends, including on failure.

When using the local version of this skill, where Local Memory is defined, default to that location for recall reads. For any version of this skill where the local memory is not available or defined, use the Google Drive plugin to obtain the memory folder. When using Google Drive, assume the memory is in a `remote-personal-memory-system.zip` file at the root level of the user's connected Google Drive account. If the user does not have a Google Drive plugin configured, or the zip file is not in the expected location, stop and ask the user what to do.

1. The input is the topics to recall. Use the appropriate configured location as described above. If no topics are given, stop and report what is missing. `<memory-location>` refers to the memory folder root. When local, that is the Local Memory path. When using Google Drive, that is the extracted zip root.
2. Read and follow `<memory-location>/always.md`.
3. Perform the Health Check below, then continue recall using the available memory.
4. Memory pages live under `<memory-location>/memory/`. Their YAML front matter contains `title`, optional `aliases`, and optional `parent` (a path relative to `memory/`). A page with a parent is a facet of that page; other relationships are links in its body.
5. For each requested topic, search memory page titles, aliases, and body text case-insensitively. Search again using aliases found and obvious alternative names or spellings. Collect matching pages.
6. For each matching page, also collect its parent chain and its facets, including nested facets, using the `parent` relationships.
7. Read every collected page in full.
8. Use the pages as working context for the current chat. Treat dated information as what was known then, not automatically as current fact. When an answer depends on what is true now, verify against a current authoritative source; if verification is unavailable, state that the remembered information may be outdated.
9. Briefly report the topics loaded and any topics without a match. The context is now loaded; do not repeat all of it unless the owner asks.

## Health Check

Do not acquire a lock or make repairs. In either case, if a problem is found, end the final recall response with `Personal memory system health:` followed by a brief description, its effect, and a suggested next step.

### 1. Access to the worker machine's local memory folder

Perform these checks against the configured `Local memory` folder, using its `operations/shared/shared-concepts.md` to interpret configuration and state:

- **Black box:** Use recent source activity and Git history to check that eligible conversations from every configured source are being successfully memorized and pending work is progressing without falling unreasonably behind, allowing for inactivity windows, schedules, quiet periods, and backfill.
- **White box:** Check queue/backfill progress, `.lock` age, Git status, and the configured backup's presence and freshness. Account for active writers; use committed state for files they may be changing. Inspect scheduler/run records only when needed to investigate a suspected problem.
- **Google Drive publishing:** If `google_drive_publishing` is `true` in `config.json`, download and safely extract the snapshot, then compare its file paths and contents with committed local `memory/` and `always.md` at one `main` revision, allowing for the publishing interval. Ignore ZIP metadata, which can change after unrelated commits. Do not upload or repair anything; report unavailable cloud verification.

If the checks pass, end the final recall response with `Personal memory system health: OK`. Otherwise, explain which checks could not be completed.

### 2. No access to the worker machine's local memory folder

Skip health checks. End with `Personal memory system health: checks require access to the worker machine.`
