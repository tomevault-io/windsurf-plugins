---
trigger: always_on
description: Turn thinking artifacts (conversation transcripts, brainstorm notes) into beautiful visual thought evolution maps. Capture, visualize, peer-read articles, and list your thinking process.
---




# WaveMind: Thought Capture + Visualization

Transform thinking artifacts into beautiful visual maps of how your ideas evolved.

## Commands

### `/wavemind capture [filepath]`
Capture a thinking artifact. Two modes:

**Mode 1: Import existing file** (`/wavemind capture <filepath>`)
1. Read the file at `<filepath>`
2. Analyze the content to extract: title, round count, word count, source type
3. Generate a short ID from the date and title (e.g., `20260327-zai-prep`)
4. Run `bash lib/capture.sh <filepath> "<title>"` to copy and index it
5. Report what was captured

**Mode 2: Live capture** (`/wavemind capture` or `/wavemind capture "Topic Name"` or `/wavemind capture "Topic Name" --output /path/to/file.md`)
When no filepath is given (or filepath doesn't exist as a file to import), start a live capture session:
1. Ask the user for a topic name if not provided
2. **Determine output path:**
   - If `--output <path>` is specified, write to that exact path. Skip folder selection and naming convention. This allows other skills (e.g. dadclawd) to call WaveMind and control where the conversation is recorded.
   - If no `--output`, pick the right folder based on topic:
     - Interview prep conversations → `~/Documents/lily-memory/Thoughts/interview_preps/<id>.md`
       Interview prep = mock interviews, interview gameplan, company research for a specific role, day-plans whose primary focus is interview prep, course-selection conversations for interview prep, pre-interview check-ins.
     - Everything else → `~/Documents/lily-memory/Thoughts/artifacts/<id>.md`
3. **Naming convention (when no --output):** `YYYYMMDD-<topic-slug>.md`. Use the format `YYYYMMDD` with the date the conversation happened. For interview prep, prefer `YYYYMMDD-interview-<slug>.md` or `YYYYMMDD-interview-study-<day>.md` / `YYYYMMDD-interview-plan-MMDD.md` over generic names like `thursday-thinking` or `plan-apr17`. The word "interview" in the slug makes the file self-describing.
4. Create the artifact file immediately at the chosen path with this structure:
   ```markdown
   # Topic Name - YYYY-MM-DD

   **Participants:** Lily + Jackie
   **Topic:** Brief description

   ## Promises
   <!-- Updated as promises emerge from conversation -->

   ---

   ## Round 1: Title
   **Lily:** ...
   **Jackie:** ...
   ```
5. **Run `date "+%Y-%m-%d %H:%M %A"` at the start of every capture session.** Record the start time in the artifact header. Run `date` again when wrapping up to calculate session duration. Never estimate times.
6. Tell the user: "Recording. Talk naturally. Say 'done' or 'save' when finished."
7. **Write-first flow:** Write both the user's words AND your response directly to the artifact markdown file first. Then give a short terminal reply (1-2 sentences max, e.g. "Written to artifact, Round 3. [brief pointer to what you said]"). Never write a full response in terminal AND in the file. The file is the single source of truth. This avoids duplication and rephrasing.
8. **Capture incrementally, not at the end.** After each round (a topic reaches a natural pause, the user moves to a new question, or a decision is made), append that round to the artifact file right away. Each round gets:
   - A section header: `## Round N: Title`
   - The raw dialogue with `**Speaker:**` labels
   - Original words preserved (including mixed languages). Fix obvious typos but do not rewrite or summarize.
   - This avoids the lossy "reconstruct everything from memory at the end" problem.
   - **Images:** When the user shares a screenshot or image during the conversation, save it to `~/Documents/lily-memory/Thoughts/artifacts/` with a descriptive filename (e.g., `milli-cafe-session.png`) and ensure the file has readable permissions (`chmod 644`). Embed it in the artifact markdown using standard markdown syntax with explicit relative path: `![description](./filename.png)`. Always use the `./` prefix so both VS Code and Obsidian can render the image. Place the image reference in the dialogue at the point it was shared, under the speaker who shared it.
   - **Promises:** When promises/action items emerge from the conversation (decisions made, tasks identified, next steps agreed), update the `## Promises` section at the top of the file. Use `- [ ] [MMDD-N]` format with IDs so the `/promise` skill can auto-extract them. Group by category if natural. Promises are a living section that grows as the conversation progresses, not something reconstructed at the end.
9. When the user says "done", "save", or "stop recording":
   - Append any remaining conversation not yet written
   - Do a final pass on the Promises section to make sure all promises from the conversation are captured with IDs
   - Run `bash lib/capture.sh` to finalize and index it
   - Report: artifact ID, title, round count, word count, file path
   - Suggest: "Run `/wavemind visualize <id>` to generate the visual."

### `/wavemind peer-read <filepath>`
Read an article together with the user, section by section, exchanging insights and building understanding through dialogue.

**Steps:**
1. Read the article at `<filepath>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lilyzhng/WaveMind](https://github.com/lilyzhng/WaveMind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
