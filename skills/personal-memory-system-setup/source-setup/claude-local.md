# Claude Local Source Setup

This setup creates an AI cron job to periodically pull chats from the Claude local internal source folder into the memory system's staging area. The AI cron job can use any AI tool located on this computer.

1. Confirm the absolute source folder where the Claude local chats are stored (usually `~/.claude/projects`). Call it `<source-folder>`
2. Record onboarding time as `<onboarding-start>` in UTC.

Use the desired AI tool's scheduler to create the AI cron job, following its [recipe](../schedulers/):

- Job name: `Personal Memory Source Staging - Claude (local)`
- Prompt: below, with the placeholders filled in (`<memory-folder>`, `<source-folder>`, and `<onboarding-start>`)
- Schedule: every two hours
- Timezone: user timezone
- Model: Suggested: Claude Haiku 5.5
- Reasoning effort: Max

## Job prompt

> Follow `Acquire Lock` in `<memory-folder>/operations/shared/shared-concepts.md`. If it returns `false`, stop.
>
> Record the current UTC time as `AI_CRON_START_TIME`.
>
> Read the `claude-local` source entry’s `staged_through` in `<memory-folder>/config.json`. Use it as `EARLIER_COPY_TIME`, or use `<onboarding-start>` if it is `null`.
>
> Find all `*.jsonl` files directly inside each project subfolder of `<source-folder>` (exactly one level below `<source-folder>`) that have a file modification time >= `EARLIER_COPY_TIME`. Do not include files in deeper subdirectories, such as `<sessionID>/subagents/agent-*.jsonl`. Gather these qualifying files.
>
> Remove from the list of qualifying files any files whose first record with `type` set to `user` has `turnOrigin` set to `scheduled`.
>
> For each final qualifying file, copy it to `<memory-folder>/source-staging/claude-local/`, ensuring the following:
>   * The file name, in the destination folder, is set to `<sessionID>.jsonl`, where `<sessionID>` is the source file name without `.jsonl` (not including `.jsonl`)
>   * The file modification time, in the destination folder, is set to `AI_CRON_START_TIME`
>   * If a file with the same name already exists at the destination, overwrite that old file with the new qualifying file, ensuring the file modification time is now `AI_CRON_START_TIME`
>
> After the entire batch succeeds, set this source entry’s `staged_through` to `AI_CRON_START_TIME` in `config.json`.
>
> Follow the `Commit Changes` section of `<memory-folder>/operations/shared/shared-concepts.md`.
> Release the lock using the `Release Lock` section of `<memory-folder>/operations/shared/shared-concepts.md`.
