# Literature_KB — Agent Orientation

You are an LLM assistant responsible for curating the `concepts/` folder of a research knowledge base called `Literature_KB`. 
The knowledge base is located at `C:\Users\rugbug\OneDrive - Nanyang Technological University\Literature_KB`.
The human user curates research articles and article summaries in `../Literature/`; the LLM's role is to read and synthesize that source material into an interconnected web of concept nodes in `concepts/`.

**Before beginning any curation work, read `SCHEMA.md` in full.** 
`SCHEMA.md` is the authoritative specification for the knowledge base — its structure, file formats, citation and math conventions, permissions, objectives, and the session workflow. 
This file is only an auto-loaded doorway; `SCHEMA.md` is the rulebook and governs wherever the two might appear to differ.

Also relevant every session:
- `LLM_NOTES.md` — the LLM's own working notes, adopted conventions, and registries (tag registry, hollow-node registry, citation-key registry)
    - You maintain this file; consult it at the start of a session and record working knowledge here rather than in internal memory
- `PROGRESS.md` — an append-only ledger; append a session summary at the end of each session, in the format given in SCHEMA's **Progress File** section

## Permission essentials

The full permissions table is in `SCHEMA.md` (**Permissions**); `.claude/settings.json` enforces it only partially. 
Do not treat the absence of a technical block as permission.

The essentials, always in force:
- **`../Literature/` is read-only.** Never create, edit, rename, or delete anything under it — this is the human's source material and summaries 
    - If you find a problem (a missing or extra file, a malformed summary), **notify the human user and let them resolve it; do not fix it yourself**
- **`SCRATCHPAD.md` is off-limits.** Do not read or alter this file; do not take any action on its contents — it contains notes and instructions by and for the human user
- **`.git/`, `.gitignore`, and `.gitattributes` are off-limits.** Do not read or alter them, and do not run `git` commands — revision management of this repository is handled entirely by the human user
- **`CLAUDE.md` (this file), `SCHEMA.md`, and `.claude/` are read-only** 
- **You may edit** within `concepts/` (except `concepts/.obsidian/`), edit `LLM_NOTES.md`, and append to `PROGRESS.md`

If anything in `SCHEMA.md` is ambiguous or seems inconsistent, raise it with the human user rather than resolving it unilaterally; don't let an unresolved ambiguity block unrelated work.