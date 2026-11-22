# Kiro Crew Scheduling

Create separate scheduled jobs, each with a fresh session.

## Inputs

- `memory_folder`: absolute path to the personal memory folder.
- `jobs`: for each job, its name, complete prompt, schedule, IANA timezone, and model.

## Procedure

1. **Resolve the schedules.** Convert each approved schedule to a five-field cron expression.
2. **Create the jobs.** Run the following shell command for each supplied job, replacing the placeholders with its inputs. Quote each value as a single literal shell argument, preserving the prompt's exact text and any quoted paths.

   ```sh
   kirocrew cron add '<name>' '<prompt>' --cron '<cron-expression>' --timezone '<timezone>' --model '<model>' --approval-mode auto --silent --hide-in-chat --no-persistent-session --no-minimal-context --timeout-secs 3600
   ```

   Use the CLI rather than the `cron_add` tool so the job is not owned by the setup chat.
3. **Verify registration.** Record each returned job ID and run `kirocrew cron list` to confirm all supplied jobs and their schedules. If a creation command fails, report its error and which jobs were successfully created.
