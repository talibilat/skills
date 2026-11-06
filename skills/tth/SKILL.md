---
name: tth
description: Time to help. Lists every skill installed across Claude Code, Codex and Cursor on this machine, read from the real SKILL.md files, with the name to type and what each one does. Use when the user types tth, asks which skills exist, what a skill does, or which skill fits a task.
---

# tth (time to help)

Answer from the files on disk and from this session's own skill list. Every row you show comes from one of those two sources.

## Steps

1. Run the scanner:

   ```
   python3 -I ~/.agents/skills/tth/scripts/skills.py
   ```

   Add `--project <repo path>` for each repo the user is working in, so project skills show up too. Add `--agent claude|codex|cursor` to show one agent, or `--find <word>` to search. `--json` gives the raw records.
   Done when the command printed its table and the count line.

2. Add the built-ins. Claude Code ships some skills inside the app (for example `code-review`, `simplify`, `loop`, `schedule`, `init`). They have no SKILL.md on disk, so the scanner cannot see them. Compare the skill list in this session's system context with the scanner output, and add each skill the session lists but the scan missed as a row marked "built-in (from this session)". Skip this step when the session lists no skills.
   Done when every skill in the session's list appears in the answer.

3. Answer the question asked:
   - "What skills do I have?": show the tables grouped as the scanner groups them, then one line with the totals per agent.
   - "Which skill for X?": run with `--find`, read the SKILL.md of the two or three best matches, and recommend one with the exact name to type.
   - "What does skill Y do?": read its SKILL.md and explain it in a few plain sentences, with an everyday example.

## Rules

- Count a skill once even when several folders link to the same file. The scanner already does this by following symlinks.
- Name collisions are real: `code-review` in `~/.agents/skills` differs from Claude Code's built-in `code-review`, and plugin skills carry a prefix (`pstack:tdd` is not `tdd`). Show the prefix.
- A skill marked "you only" has `disable-model-invocation: true`. Only a person typing its name can start it.
- If the scanner fails or a location is missing, say which one and why. Never fill a gap from memory.

Where each agent loads skills from, and how that was checked, is in `references/locations.md`. Read it when a count looks wrong or a new agent is added.
