---
name: personal-memory-system-offboard-source
description: Stop collecting a selected conversation source in an existing personal memory system while retaining its captures and memories.
---
# Personal Memory System Offboard Source

Inputs:
  1. `<memory-folder>` path
  2. Source type to offboard

Definitions:
  1. `<shared-concepts>` refers to `<memory-folder>/operations/shared/shared-concepts.md`

## Confirm the Scope

1. Resolve the memory folder to an absolute path. Read its `config.json` and the `Memory Configuration`, `Source Type Definitions`, and `Scratch` sections of `<shared-concepts>`. Ask the owner to select or confirm one source identifier and all its configured locations.
2. Explain that offboarding stops future collection, not forgetting: existing captures, memories, staged files, Library exports, and Git history remain. Already saved queue and backfill snapshots remain scheduled for normal memorization, so memories can still change after offboarding. Ask the owner to approve this scope before proceeding. If they want pending ingestion stopped or stored data removed, resolve that as a separate request rather than silently changing this default.

## Stop Source Jobs

1. Locate the installed `personal-memory-system-setup` skill and read the selected source's recipe in `source-setup/`. Use judgement to plan every job that must be disabled.
2. Disable only jobs dedicated to this source. Do not stop shared discovery, memorization, troubleshooting, backup, or Google Drive publishing jobs.
3. For jobs that are owned by another tool or cannot be automatically deleted, work with the owner to remove them.

## Remove from Configuration

1. Follow `Acquire Lock` in `<shared-concepts>`; if it returns `false`, stop.
2. Remove only the selected source entry from `sources`, preserving every other source and setting. Keep its installed source definition and all stored data. Leave the queue, backfill worklist, snapshots, and discovery watermark unchanged so pending ingestion can continue normally.
3. Follow `Commit Changes`, then `Release Lock` in `<shared-concepts>`.
