---
name: personal-memory-system-memorize
description: Memorize the oldest queued conversation snapshot.
---
# Personal Memory System Memorize

Inputs:
  1. `<memory-folder>` path

Definitions:
  1. `<shared-concepts>` refers to `<memory-folder>/operations/shared/shared-concepts.md`

1. The input to this skill is the path to the personal memory folder. This is what `<memory-folder>` refers to in this skill. If missing from input, stop here. Follow `Scratch` in `<shared-concepts>`.
2. Follow `Acquire Lock` in `<shared-concepts>`. If it returns `false`, stop.
3. Read the queue using the `Queue` section of `<shared-concepts>`. If its `items` array is empty, release the lock and stop.
4. Select the oldest entry in `items` as `<selected-item>`.
5. Open `<selected-item>`’s `path` relative to `<memory-folder>` as `<selected-snapshot>`. Read its source type, UID, source modification time, and hash from the filename using the `Capture` section of `<shared-concepts>`.
6. Set `<capture-destination>` to `<memory-folder>/captures/` followed by the filename of `<selected-snapshot>`.
7. Find the most recent capture in `<memory-folder>/captures/` for the same source type and UID, ordered by the source modification time in its filename. Call it `<latest-capture>`.
8. If `<latest-capture>` exists and the incoming snapshot has an earlier source modification time or the same hash, delete the `<selected-snapshot>` file. Otherwise, move `<selected-snapshot>` to `<capture-destination>`.
9. If the snapshot was deleted in step 8, skip this step. Otherwise, read [Memory Extraction](#memory-extraction) below and the `Memory Pages` section of `<shared-concepts>`. Apply the matching case:
    1. **No previous capture:** Memorize `<capture-destination>` in full.
    2. **Previous capture exists:** Memorize only the delta from `<latest-capture>`, using the full conversation for context.
10. Remove `<selected-item>` from the queue’s `items` array.
11. Follow the `Commit Changes` section of `<shared-concepts>`.
12. Release the acquired lock using the `Release Lock` section of `<shared-concepts>`.

## Memory Extraction

You MUST finish the required extraction for the entire capture or delta in this run, regardless of its size or the time required. Do NOT voluntarily stop execution after partial review or defer remaining work.

Complete extraction of all accessible source content. Recover readable content from truncation or other recoverable read failures and continue. Unavailable content does not block processing the rest: preserve its references, never invent its contents, and report the gap only if it may materially affect the memory. Do not stop merely because some content cannot be accessed.

Tool-output truncation is a per-call display limit, not a capacity blocker. Recover omitted content with smaller bounded reads until the entire required conversation, including tool calls and results, has been reviewed. Track coverage across reads and compactions. Never infer a blocker from file size, record count, or truncated output alone. Finish updating and verifying the memory pages before reporting completion. If the owner stops the run or a genuine unrecoverable execution or access limit prevents completion, report the limitation and unfinished coverage truthfully.

Never modify `<memory-folder>/always.md`; it is owner-managed.

Preserve enough understanding that the owner can leave any chat and start a fresh one without fearing the loss of valuable context or having to explain everything again.

Weigh each claim by what supports it: the owner’s statements or confirmations, original source material, tool results, or an AI assistant’s assertions. An assistant’s assertion alone does not establish a fact; it needs the owner’s confirmation or supporting evidence from the conversation. Keep unsupported assistant claims unconfirmed.

In cloud chats created by dot, use delegation metadata to identify dot’s prompts as assistant-authored, even when labeled as user messages. Distinguish dot’s claims and instructions from quoted owner statements.

Preserve useful beliefs, preferences, intentions, and estimates as statements attributed to whoever expressed them. They need not be objectively proven to be worth remembering; retain their uncertainty rather than presenting them as established facts.

Read the complete captured conversation, including tool calls and results. Retain context that would help a future chat understand the owner’s personal and professional life, make decisions, continue effectively, or avoid rediscovery and repeated explanation: the owner’s goals, preferences and constraints, decisions and their reasons, useful rejected approaches and failed attempts, working knowledge, current state, unresolved questions, and next actions.

The `chatgpt-dot` source comes from dot, a persistent personal assistant whose ongoing conversation is exported in chunks rather than as separate complete chat sessions. Each capture is a partial continuation, not a full-session snapshot. Memorize only the current chunk. Use earlier captures from the same room as context to interpret its messages, locating them by the room ID and time bounds in their headers rather than by source UID. Do not re-ingest those earlier captures; absence of earlier context in the current chunk is not evidence of a contradiction or retraction.

Before writing, search existing memory pages for the conversation’s subjects, names, aliases, and key terms. Read each matching page in full before deciding what to change. Update relevant existing memory pages before creating new ones, integrating new information without repeating what is already known. Clearly distinguish what happened, what was decided, what is intended or proposed, what was attempted, and what remains unresolved. Record each at the level the evidence supports; a plan is not a completed action, and an attempt is not a confirmed outcome. Preserve meaningful dates and uncertainty. Organize by the subjects discussed rather than by chat, and create no page when the conversation adds no useful information. Omit chatter and process narration that add no useful context.

Include useful web links and links to related memory pages or specific sections.

Support memorized information with relative links to its captured conversation and the relevant record identifiers or line numbers (call this an "anchor"). Preserve useful web links from the conversation. Never edit or overwrite a saved capture.

When renaming, moving, or merging memory pages or their sections, update affected links and `parent` references, including relative links within moved pages. Parent relationships must not form cycles.

IMPORTANT: Preserve previously memorized information. When information changes, place the previous information under a `## Superseded` heading, retaining its dates and supporting sources and identifying what replaced it. Make the current understanding clear. The journey should be easily understandable.

After writing, reread each changed page in full and check for contradictions, including claims supported by other conversations. Move information shown to be outdated to Superseded; when the evidence does not resolve a conflict, preserve the competing claims with their sources and make the disagreement explicit. Before committing, inspect the diff: for every removed passage, locate its information, dates, and sources in the revised text, another memory page, or Superseded. After a failed patch, verify the retry preserves the intended semantic change without unrelated changes.

Compare each message’s date with the current date when judging whether its information is still current; one conversation may span a long period. Use reasonable judgment rather than a fixed age cutoff: durable preferences and knowledge may remain useful for years, while temporary plans and project status can become stale quickly. Record dated information as historical when appropriate; do not assume it is current just because it is newly memorized, or obsolete just because it is old. Use the dates of events and statements to reconcile it with existing memory, preserving uncertainty when current relevance is unclear.

For time-dependent information, record when it happened or was true using an explicit date from the conversation, or the relevant message’s timestamp when it establishes the state at that time.
