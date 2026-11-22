# Claude Desktop Scheduling

Have the owner create separate scheduled tasks in the Claude desktop app, each run in a fresh session.

## Inputs

- `memory_folder`: absolute path to the personal memory folder
- Job name
- Prompt
- Schedule
- Timezone
- Model
- Reasoning effort

## Procedure

1. **Give the task settings.** For each supplied job, give the owner exactly what to enter when creating a new scheduled task in the app's Scheduled section:

   | Setting | Value |
   | --- | --- |
   | Name | Supplied job name |
   | Prompt | Supplied complete prompt, ready to paste |
   | Working folder | `<memory_folder>` |
   | Schedule | Supplied schedule in the supplied timezone |
   | Model | Supplied model |
   | Reasoning effort | Supplied effort, if offered |
   | Permission mode | Auto, so runs proceed unattended |

2. **Verify registration.** Wait for the owner's confirmation, then run `list_scheduled_tasks` to confirm each supplied job exists with the expected schedule and `model`. Report any missing job or mismatch.
