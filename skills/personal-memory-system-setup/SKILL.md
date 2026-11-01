---
name: personal-memory-system-setup
description: Begin personal memory setup by asking which supported AI tools' conversations the owner wants remembered.
---
# Personal Memory System Setup

## Ensure Skills Are Installed

1. Check that every personal-memory-system skill from the same source as this setup skill is installed and available to the current AI tool. Install or update any missing or outdated skills. Defer skills that require configuration until their setup steps below. Preserve configured values when updating installed skills.

## Choose Sources

1. Read every Markdown file directly in [source-types/](source-types/). Each file defines one available source type. Read its **Identifier** and **Display Name**. Offer all supported sources.
2. Ask: “Which AI tools' conversations do you want this system to automatically learn from? You may skip source selection for now.” Show the available sources and allow any number of selections. Use an interactive question tool if it supports multiple selections; otherwise ask in chat.
3. Wait for the owner's answer and confirm the selected sources.

## Create the Memory Folder

1. Ask: “Where would you like this system to store your personal memory? Default: `~/Desktop/personal-memory-system`.”
2. Wait for the owner's answer and confirm the storage location.
3. Resolve the approved storage path and create the folder with `mkdir` without `-p`. Fail and stop if anything already exists at that path, including an empty folder, file, or symlink. Never reuse or overwrite it.
4. This folder name, going forward in this skill, is `<memory-folder>`. Follow [Scratch](shared-concepts.md#scratch).
5. Create `captures/`, `memory/`, and `operations/queue-snapshots/` under `<memory-folder>`. Add an empty `.gitkeep` file in each so Git preserves the folders during backup and recovery.
6. Create `source-staging/` under `<memory-folder>` for source input staging as defined in [Source Staging](shared-concepts.md#source-staging). Do not add a `.gitkeep` file there.

## Choose Backup Folder

1. Ask: “Would you like to back up your memory to a cloud-synced folder, such as iCloud Drive, OneDrive, or Dropbox, so it can survive losing this computer? If yes, which folder should hold the backup? You may skip backup.”
2. Wait for the owner's answer. If they choose backup, expand the folder to an absolute path and confirm it exists, is outside `<memory-folder>`, and is synced by their cloud client. Ask the owner to confirm cloud syncing if it cannot be determined locally.
3. If they chose backup, form the full backup file path and extension using the [Memory Configuration](shared-concepts.md#memory-configuration) definition and show its path to the user. If that file already exists, ask for a different folder to avoid overwriting an existing backup.

## Choose Google Drive Publishing

1. Ask: “When you are using AI tools that are not local to this computer, they will not be able to access the memory we are setting up. To allow for remote AI tools to utilize memory, would you like to publish committed memory as a private `remote-personal-memory-system.zip` in your connected Google Drive account’s My Drive root?” Wait for the owner’s answer.
2. If the user accepts, work with them to enable the AI tool's Google Drive plugin, and note down their acceptance for later.

## Save Configuration

1. Create `<memory-folder>/config.json` using the [Memory Configuration](shared-concepts.md#memory-configuration) definition. Initialize `sources` as an empty array; source onboarding below adds verified sources. Set `backup_file` from their choice and initialize `google_drive_publishing` to `false`; publishing setup below enables it. Tell the owner where it was saved and show it inline.

## Install Recall

1. Prepare the recall skill from the same source as this setup skill, replacing `<local-path>` with the absolute `<memory-folder>`. Install this configured copy for the current AI tool.

## Install Shared Concepts

1. Create `<memory-folder>/operations/shared/` with `mkdir -p`, then copy [shared-concepts.md](shared-concepts.md) into it.
2. Copy [source-types/](source-types/) into `<memory-folder>/operations/shared/source-types/`.

## Install Backup

1. If the owner chose backup, copy [scripts/backup.sh](scripts/backup.sh) to `<memory-folder>/operations/backup.sh` and make it executable.

## Install Always

1. Explain that `always.md` tells agents how to load standing instructions, with local exceptions under `Personal Adjustments`. Recommend a small index page or Markdown file that links to separate instruction pages, each covering one topic or phase. Each index entry should say when it applies; mark general rules as applying to all work. This lets agents read the relevant pages without loading the entire collection.
2. Ask for the URL or local file path to that index and any personal adjustments to include. The owner may create an index now or leave its location and adjustments for later. Wait for their answer.
3. If an index was supplied, read it and check that its entries describe when they apply and link to instruction pages that can be read separately. If that structure is missing, explain what needs changing to the owner.
4. Copy [always.md](always.md) to `<memory-folder>/always.md`. If an index was supplied, replace `<agent-instructions-location>` with its URL or local file path, expanding local paths to absolute paths. Otherwise, remove the entire `Agent Instructions` section, including its subsections. Replace `<personal-adjustments-here>` with the supplied adjustments, or remove the placeholder if none were supplied. Show the resulting file to the owner; they maintain it going forward.

## Initialize Queue and Backfill

1. Create the initial queue file using the [Queue](shared-concepts.md#queue) definition, with `discovered_through` set to the current UTC time minus two hours and an empty `items` array.
2. Create the backfill worklist using the [Backfill](shared-concepts.md#backfill) definition, with an empty `items` array.

## Initialize Git

1. Initialize the new memory folder as a Git repository with `git init -b main`.
2. Create `.gitignore`, and add `.lock/` and `.DS_Store` to it.
3. Run `git add .` from `<memory-folder>` and commit the initial state.
4. Confirm that the commit succeeded and report the result to the owner. If initialization or the commit fails, stop and report the error.

## Onboard Selected Sources

1. If the owner skipped source selection, skip this section. Otherwise run `$personal-memory-system-onboard-source` once for each selected source, supplying `<memory-folder>` and that source.
2. After onboarding succeeds, ask the user if they would like to backfill old captures from any onboarded source. Explain that current onboarding will not capture past items. If the user would like to backfill, run `$personal-memory-system-backfill` for that source(s).

## macOS Permissions for Scheduled Scripts

If scheduled backup below fails with `Operation not permitted`, ask the owner to check **System Settings → Privacy & Security → Full Disk Access** for `/bin/bash`. Explain that this grants access to scripts run through Bash; let the owner enable it if they choose, then retry the affected scheduled job and verify it.

## Schedule Backup

1. If `backup_file` is `null`, skip this section. Otherwise ask: “How often should the backup run? Recommended: every hour.” Wait for the owner's answer and confirm the interval.
2. Fill in [scripts/backup.plist](scripts/backup.plist) with the job label, absolute paths, and approved interval; install it in `~/Library/LaunchAgents/` and load it with `launchctl`.
3. Trigger the job once through the scheduler and wait for it to finish. Confirm a successful exit and verify the destination bundle with `git -C "<memory-folder>" bundle verify "<backup_file>"`. Report the backup path, schedule, and job identifier.

## Set Up Google Drive Publishing

1. If publishing was declined, skip this section.
2. Create the ZIP described in the job prompt below. Upload it once as `remote-personal-memory-system.zip` to the connected account’s My Drive root. Set `google_drive_publishing` to `true` and commit the configuration locally.
3. Use the desired AI tool's scheduler to create the AI cron job, following its [recipe](schedulers/):
  - Job name: `Personal Memory System Google Drive Publishing`
  - Prompt: below, with the placeholders filled in (`<memory-folder>`)
  - Schedule: every hour
  - Timezone: user timezone
  - Suggested Model: GPT-6 Luna
  - Suggested Reasoning Effort: High
4. Let the user know that they can now use the memory system with any AI tool that has access to Google Drive, and all they need to do is supply it the recall prompt (they can feel free to put it in their Google Drive, install it in their other remote AI tool, give the desired chat the link to the Github repository's copy of recall, and so on).

### Google Drive Publishing Job Prompt

> Read `<memory-folder>/operations/shared/shared-concepts.md` for context. Create a temporary ZIP containing only `memory/` and `always.md` from the committed `main` branch of `<memory-folder>`. Using the Google Drive plugin connector, replace the file `remote-personal-memory-system.zip` in My Drive’s root. Remove the temporary ZIP when done.

## Schedule Discovery, Memorize, and Troubleshoot

1. Ask: “When should discovery, memorization, and troubleshooting run? Recommended: troubleshooting hourly at minute :00, discovery hourly at minute :10, and memorization every hour at minutes :20 and :45, in your local timezone.” Explain that this gives discovery 10 minutes before memorization starts, and each memorization run 25 minutes before the next discovery or memorization run. Wait for the owner's answer and confirm all three schedules and the timezone. Explain that troubleshooting checks locks at least one hour old, leaves active runs and uncertain cases unchanged, and discards uncommitted work only after confirming abandonment.
2. Ask which model and reasoning effort to use for each of discovery, memorization, and troubleshooting. Wait for the owner's answer and confirm all three choices.
3. Follow the current AI tool's recipe in [schedulers/](schedulers/), supplying the absolute `<memory-folder>` path and the approved schedules, timezone, models, reasoning effort, and local execution for these jobs:
   - **Personal Memory Discover:** `$personal-memory-system-discover "<memory-folder>"`
   - **Personal Memory Memorize:** `$personal-memory-system-memorize "<memory-folder>"`
   - **Personal Memory Troubleshoot:** `$personal-memory-system-troubleshoot "<memory-folder>"`

## Explain How to Use Your Memory

Give the owner this short guide, using their actual memory folder and configured tools:

- Start a new chat with `$personal-memory-system-recall <topics>` when you want remembered context.
- Teach through ordinary conversations: explain your preferences, decisions, reasons, and corrections. Conversations from the configured sources are processed on the schedules set up above; new information may take time to become available to recall.
- To remember something from a document, website, or other source, ask the chat to read it and discuss what matters. The material must appear in the conversation or its tool results; sharing a link alone does not capture its contents.
- Keep standing instructions in the resource linked from `always.md`, with personal exceptions in that file. You maintain these instructions; memorization does not update them.
- Use recall to start fresh chats as needed. If something important is missing or wrong, explain it in a configured chat so a later processing run can incorporate the correction.
