# Literature_KB Schema

This file should be read in its entirety before any other file.
This file describes the role and responsibilities of the LLM for a research knowledge base and the structure and conventions of the knowledge base directory.
The knowledge base spans machine learning research, concentrated most heavily at present in the AI subfields of self-supervised learning, generative models, and adversarial examples.

For context, there are multiple parties and programs involved in the curation of this knowledge base:
- Microsoft OneDrive — Hosts the knowledge base folder and synchronizes it across devices
- Human User — Curate research articles and summaries and utilize the knowledge base for research insights
- LLM — Analyzes research articles and summaries and curates a system of interlinked concept node files 
- Obsidian — Visualizes the knowledge graph of concepts by connecting concept nodes via their wikilinks

---

## LLM Role and Responsibilities

### Purpose

The `concepts/` folder exists to transform a large collection of individual article summaries into an interconnected, queryable knowledge base. 
It serves to:
- **Organize** — summarize and centralize research findings, methods, and trends into digestible concept nodes that draw from multiple articles
- **Synthesize** — identify how findings across works interact, agree, conflict, or build on each other, making relationships between ideas explicit
- **Illuminate** — surface open questions, underexplored directions, and opportunities to guide ongoing and future research
- **Situate** — track how concepts have evolved historically, identifying foundational works and shifts in the field's thinking over time
- **Standardize** — identify recurring methodological patterns, datasets, and evaluation metrics that reflect community consensus on best practice
- **Visualize** — curate an interconnected system of concept nodes using concept node files and wikilinks to facilitate knowledge graph visualization

### Responsibilities

In each session, the LLM is expected to:
- Read `SCHEMA.md` before any other file
- Create and maintain concept nodes as `.md` files in `concepts/` by reading and synthesizing from `../Literature/`, guided by the objectives defined in **Purpose**
- Actively check for new or updated articles in `../Literature/` that are not yet reflected in any concept node file, and incorporate their information appropriately
    - Be aware that the human user may add or edit articles and summaries to `../Literature/` over time
- Create and maintain the network of wikilinks within concept node files that form the edges between nodes of the knowledge graph, guided by the objectives defined in **Purpose**
- Frequently review and update the set of wikilinks within each concept node file to ensure it accurately and adequately references related concepts
- Record working notes and adopted conventions in `LLM_NOTES.md` rather than in internal memory, as described in **LLM Notes File**
- Append a session summary to PROGRESS.md at the end of each session, as described in **Progress File**
- Adhere to the file and folder permissions defined in **Permissions**

If any instruction in this file is ambiguous, appears internally inconsistent, or conflicts with what you observe in the knowledge base, raise it with the human user rather than resolving it unilaterally. 
Do not let an unresolved ambiguity block work that does not depend on it — proceed with the rest of the task, state the assumption you have made in the interim, and record the issue in the **Issues Flagged** field of the session's `PROGRESS.md` entry.

---

## Knowledge Base Structure

At instantiation, the knowledge base is structured as follows:
- `SCHEMA.md` — This file
- `CLAUDE.md` — An auto-loaded orientation file that directs the LLM to this specification; maintained by the human user
- `SCRATCHPAD.md` — Personal notes by the human user
- `LLM_NOTES.md` — The LLM's own working notes and conventions, kept visible to the human user; see **LLM Notes File**
- `PROGRESS.md` — An append-only ledger of session summaries; see **Progress File**
- `.claude/` — Contains the `settings.json` file with partial programmatic enforcement of the permissions below
- `concepts/` — Contains LLM-synthesized concept nodes; see **Concept Curation**
    - `.obsidian/` — Obsidian's configuration directory, inside the `concepts/` vault root and maintained entirely by Obsidian
- `../Literature/` — The external article source corpus, maintained by the human user; see **Literature Folder**
    - `obsidian/` and `Literature_DB.base` — A separate Obsidian configuration directory used by the human-user to view and sort articles

### Permissions

The LLM must follow the file and folder permissions described below.
For folders, the permitted and prohibited actions apply to all subfolders and files contained within. 
Where two rows both apply, the more specific path governs.

Actions are:
- **Read** — View contents
- **Create** — Add a new file, or overwrite an existing file wholesale
- **Append** — Add new content to the end of a file, leaving existing content untouched
- **Modify** — Change or remove content that is already present
- **Rename** — Change a file or folder name
- **Delete** — Remove a file or folder

| Folder or File | Permitted Actions | Prohibited Actions | Comments |
|---|---|---|---|
| `SCHEMA.md` | Read | Create, Append, Modify, Rename, Delete | You must not alter this file |
| `CLAUDE.md` | Read | Create, Append, Modify, Rename, Delete | You must not alter this file |
| `SCRATCHPAD.md` | — | Read, Create, Append, Modify, Rename, Delete | You must not read or alter this file; you must not take any action based on its contents |
| `LLM_NOTES.md` | Read, Create, Append, Modify | Rename, Delete | You may freely edit this file |
| `PROGRESS.md` | Read, Append | Create, Modify, Rename, Delete | Edits are append-only; you may never alter or remove past entries |
| `.claude/` | Read | Create, Append, Modify, Rename, Delete | You must not alter the files here or attempt to circumvent their intended protections |
| `concepts/` | Read, Create, Append, Modify, Rename, Delete | — | You may freely create, edit, and remove files here; `concepts/.obsidian/` is excepted below |
| `concepts/.obsidian/` | Read | Create, Append, Modify, Rename, Delete | You must not alter any folders or files here; maintained entirely by Obsidian |
| `../Literature/` | Read | Create, Modify, Rename, Delete | You may read from files here; you must not create, alter, or remove any files here |

Note that some of these permissions are enforced programmatically by `.claude/settings.json`; others depend on the LLM's compliance. 
Do not treat the absence of a technical block as permission — this file states the intent, and the settings file is a partial, best-effort enforcement of it.

---

## LLM Notes File

`LLM_NOTES.md` is the LLM's own notes file. 
Unlike `SCHEMA.md`, which is authoritative and maintained by the human user, and `SCRATCHPAD.md`, which belongs to the human user and must not be acted on, `LLM_NOTES.md` belongs to the LLM: the LLM maintains it and may rely on its contents across sessions, within the permissions defined above.
The LLM is expected to record its working notes and adopted conventions in `LLM_NOTES.md` rather than in its own internal memory, so that this working knowledge stays visible to the human user. 
Use `LLM_NOTES.md` to record material that supports curation of the knowledge graph.

Examples of possible uses for `LLM_NOTES.md` include but are not limited to:
- The LLM's own working material
    - Conventions the LLM has adopted where `SCHEMA.md` leaves a choice open
    - Living registries the LLM maintains, such as a concept tag registry (the working tag vocabulary), a hollow-node registry (each planned concept's intended focus and scope), or a reference citation-key registry (folder → key)
- Observations for the human user's later review
    - Concerns, confusions, or apparent conflicts in `SCHEMA.md`
    - Significant gaps in the article corpus itself, such as an influential work or research area not present in `../Literature/`

Records in `LLM_NOTES.md` are for the human user to review at their own pace. 
An issue needing timely attention — such as an article folder missing a file — should instead be raised directly with the human user, since `LLM_NOTES.md` may not be checked promptly.

`SCHEMA.md` is authoritative and `LLM_NOTES.md` is subordinate to it. 
Nothing recorded in `LLM_NOTES.md` overrides `SCHEMA.md`; where the two appear to conflict, `SCHEMA.md` governs. 
The LLM does not edit or disregard `SCHEMA.md`, and recording a convention or concern in `LLM_NOTES.md` does not itself change any rule — it is a note for the human user, who reviews `LLM_NOTES.md` over time and makes any changes to `SCHEMA.md` themselves. 
A convention recorded here is the LLM's own working reference only; a reader that lacks `LLM_NOTES.md` — such as a fresh session or sub-agent — will not see it unless the human user has incorporated it into `SCHEMA.md`.

---

## Progress File

`PROGRESS.md` is an append-only ledger of session summaries that carries continuity across sessions.
At the end of each session, append one new entry to the end of the file; never alter or remove past entries.
Its first entry is a filled-in template that shows the format.

Each entry opens with a date header — `## YYYY/MM/DD (Session N)` followed by a bolded `Summary:` field:
- Summary — A brief description of the session's updates and progress, limited to 150 words or less.

---

## Literature Folder

The `../Literature/` folder is the user-maintained, read-only source corpus that concept nodes draw on.
There is one folder per article, each holding an article file and the user's `summary.md`. 
This section covers how those folders are named and organized, the `summary.md` format, and how to access the article files.

### Naming

Each folder name in `../Literature/` follows the convention: PublicationYear_FirstAuthor_Title_of_Work

- Splitting on `_` gives: publication year (index 0), first author (index 1), and article title words (index 2+)
- Folder names only include alphanumeric characters, `-`, and `_`

### Contents

Each folder should contain exactly two files, `summary.md` and one of `article.pdf` or `article.md`.
File descriptions are given below:
- `summary.md` — Notes and article summary by the user
- `article.pdf` — PDFs of papers and academic articles - many of these are annotated with highlights, notes, and comments by the user
- `article.md` — Clips of blog posts obtained with Obsidian Web Clipper - these are not edited by the user

### Summary File Format

Each `summary.md` file begins with a YAML frontmatter block enclosed in `---` delimiters:
| Field | Description | Always Filled | Format | 
|---|---|---|---|
| `title` | Full article title | Yes | Freeform text |
| `type` | Article format | Yes | Label: `paper` or `blog` |
| `link` | URL to source article | No - empty for unobtainable works | Freeform text, typically in URL style |
| `models` | Names of significant models or datasets introduced by the work | No - empty implies no model or dataset | Freeform text |
| `authors` | List of author names | Yes | Freeform text |
| `date` | Article publication date | Yes | Date in `YYYY/MM/DD` format |
| `read_date` | Date article was read | No - empty implies unread | Date in `YYYY/MM/DD` format |
| `status` | Level of engagement with article | Yes | Label: `Unread`, `Skimmed`, `Part Read`, `Read`, `Part Notes`, `Noted`, or `Reviewed` |
| `task` | Intended action (for the human user) | No - empty implies no current task | Label: `Assess`, `Skim`, `Read`, `Note`, or `Review` |

Additional notes about `summary.md` YAML frontmatter:
- The user applies the `status` label consistently — it is a reliable signal of engagement
- A higher `status` generally corresponds to a more detailed summary, though this is a tendency rather than a rule
- The `task` field represents the task for the human user — the LLM should not interpret this field as a task for the LLM

Each `summary.md` file contains a freeform text summary under `## Summary` that follows the YAML frontmatter.

### Summary File Example

```
---
title: "The information bottleneck method"
type: "paper"
link: "https://arxiv.org/abs/physics/0004057"
models: ""
authors: "Tishby, Pereira, Bialek"
date: "2000/04/24"
read_date: "2024/01/28"
status: "Noted"
task: ""
---

## Summary

Freeform text summary begins here.

```

### Accessing Article Files

This subsection covers working with the article files in `../Literature/`: checking each folder's contents, opening the two article formats, and staying current as the corpus changes.

- Advise the human user of any folder in `../Literature/` that is missing a file or contains an extra file
    - Do not take any action to create, alter, or remove files here
- `article.pdf` contains the article's original content; it may also contain the user's annotations, such as highlights and typed margin notes
    - Treat the article's original content as the objective — annotations are secondary and usually clarify or restate the article rather than add independent insight
    - It is recommended to use PDF extraction tooling Poppler (installed at `C:\Tools\poppler\Library\bin\` (powershell) or `/c/Tools/poppler/Library/bin/` (bash))
        - `pdftoppm.exe` — Renders pages to images (the Read tool uses this internally)
            - Best for viewing figures, diagrams, and plots
            - Displays tables in their original layouts, without scrambling their contents
            - Displays page layout, typeset equations, user highlights, and margin notes as the user sees them
        - `pdftotext.exe` — Extracts the text layer
            - Best for viewing exact text, which may include exact author names, URLs, and LaTeX math source
            - Presents a convenient format for grepping the entire paper or reading long prose cheaply
        - If invoking `pdftotext.exe`, utilize its full path — calling bare `pdftotext.exe` will execute a different, older tool
- `article.md` contains a blog post's content; it is an unedited web clip, so it contains no user annotations
- Actively check for article folders that are new or updated, since the user may add or edit `../Literature/` over time

---

## Concept Curation

The `concepts/` folder holds the LLM-synthesized concept nodes, written as `.md` files, and is the root of the Obsidian vault that visualizes them as a knowledge graph.
This section covers how that folder is organized and how the LLM builds concept nodes from the article corpus.
The layout of an individual node is specified separately in **Concept Node File Format**.

### Organization

- One file per concept — each node corresponds to a single distinct idea, method, or theme
- Varied concept breadth — different concepts may have different granularities as necessary to maintain a coherent and logical knowledge graph
- Flat structure — no subdirectories; hierarchy and relationships emerge naturally from concept node organization and link density in the knowledge graph
- A concept may be broad enough to encompass narrower ones — linking out to them rather than absorbing their detail — while remaining one node like any other
- Specific models and datasets are not concept nodes themselves, but may feature prominently within relevant concept nodes as examples or instantiations
- File naming follows `lowercase_with_underscores.md` (e.g., `score_matching.md`)

### Concept Creation Guidelines

There is a large degree of independence granted to the LLM for the curation of the concept node files and their interlinking.
Concept node file creation, editing, and review is almost entirely the responsibility of the LLM, perhaps with minor guidance from the human user.
The LLM is permitted to add to, reduce, merge, or split concept node files at its discretion.
Any consequences from modifying concept files (e.g., broken wikilinks, incomplete concepts, overlapping concepts) should be handled by the LLM.
In general, new concept node files should be created in the following situations:
- Creation is justified by the body of existing research articles and summaries
- There exists a notable conceptual gap in the existing concept node files
- An existing concept node file is overloaded with varied information to the point that adequate conceptual clarity can no longer be maintained

Before creating a new concept node, the LLM should:
- Check whether an existing concept node file already covers the concept before creating a new file
    - Be wary of duplicating nearly-identical concepts under different filenames
- Consider whether expanding existing concept node files would maintain a more coherent and insightful knowledge graph

During creation of a new concept node, the LLM should:
- Identify the articles relevant to the concept, as described in **Identifying Relevant Articles**
- Determine the node's core focus and scope — what the concept covers, and how it is bounded from neighboring concepts
- Extract the information the node needs from those articles, as described in **Extracting from Articles**
- Consider how other concept nodes might inform the summaries and analysis contained within the new concept node file

After creating a new concept node, the LLM should:
- Consider editing existing concept nodes in response to information contained in the new concept node file
- Review and update the sets of wikilinks within other concept node files

### Identifying Relevant Articles

Identifying which article folders are relevant to a concept should stay light — the goal is to find candidates without reading each one in full. 
Relevance to the concept decides whether to draw on an article: a relevant but unread or lightly-engaged work should still be drawn on. 
Extracting information from the identified articles is deeper and is covered in **Extracting from Articles**.

Use your own judgment when selecting a method to efficiently identify relevant articles.
Multiple avenues of identifying article relevance exist, requiring different levels of effort:
- The article title is free from the folder name or low-cost from the `summary.md` YAML frontmatter
- The `models` field from `summary.md`, when it exists, is also low-cost
- The article's abstract from the first page of an `article.pdf` or the opening of an `article.md` surfaces the high-level setting and contributions
- The summary from `summary.md`, where it has substance, is more descriptive and typically of medium length
    - The `status` field reflects the user's engagement and roughly how much the summary offers
- A full `article.pdf` or `article.md` read is heavier and usually reserved for extraction but useful for gauging relevance if cheaper avenues are ineffective 
- Training knowledge of a work may serve as a lens for judging relevance, but not as a source for the node's claims, which must rest on articles actually consulted

### Extracting from Articles

Extraction is the deeper work of mining an identified article for the information a concept node needs. 
See **Accessing Article Files** for how the source files are organized and opened.
See the `## Synthesis` subsection description under **Concept Node File Format** for the kinds of information to focus on.

Use your own judgment when deciding how deeply to extract, guided by the depth the concept warrants. 
Articles and their summaries hold information at varying levels of depth and detail:
- The article's abstract from the first page of an `article.pdf` or the opening of an `article.md` surfaces the high-level setting and contributions
- If the `summary.md` file has a high `status` level, then the `summary.md` body is typically of medium detail and contains the user's own synthesis 
- A full `article.pdf` or `article.md` read is the deepest source, carrying exact results, figures, and equations

Where a relevant article cannot be fully consulted, extract what you can and note the weaker support where the claim is made, as described in the `## Synthesis` subsection description under **Concept Node File Format**.

---

## Concept Node File Format

Each concept node file should begin with a YAML frontmatter block enclosed in `---` delimiters, followed by the subsections `## Summary`, `## Related Concepts`, `## Open Questions`, `## Synthesis`, and `## References`.

### YAML Frontmatter Fields

| Field | Description | Format | Comments |
|---|---|---|---|
| `title` | Human-readable concept name | Freeform text | Does not affect wikilink resolution |
| `tags` | Concise concept labels | Freeform text | Used for sorting, filtering, and grouping nodes in Obsidian | 
| `last_updated` | Date of last file content update | Date in `YYYY/MM/DD` format | Used for edit transparency |

The LLM is responsible for designating and updating `tags`.
Reuse an existing tag rather than coin a near-duplicate, keeping the tag vocabulary clear for sorting and filtering. 
A tag registry in `LLM_NOTES.md` can serve as the working vocabulary.
`last_updated` should be updated when files receive significant content updates; minor file changes like adding a wikilink or fixing a typo do not warrant updating this field.

### Subsection Descriptions

Description of `## Summary`:
- A brief (~100 words), precise summary of the concept
- Covers what the concept is, why it matters, and its role in the broader research landscape

Description of `## Related Concepts`:
- A series of wikilinks that reference related concept node files
- Each wikilink consists of a separate concept node file name (without the `.md` extension) within double square brackets
    - e.g., to facilitate a link to the concept covered by `score_matching.md`, add wikilink `[[score_matching]]`
- There are no hard constraints on the number of wikilinks
- The number of wikilinks included in a concept node file should be sufficient to create a coherent and logical knowledge graph but not so high as to diminish the significance of a wikilink
- Wikilinks may reference concepts that do not exist yet
    - These render as unresolved (hollow) nodes in the Obsidian graph
    - Use them only to mark proposed or planned concepts for future sessions
    - A registry of hollow nodes and their intended focus and scope in `LLM_NOTES.md` can help later sessions build the nodes still wanted and prune those whose plan has lapsed
- Do not remove a wikilink solely because its target file does not exist; remove it only if the planned concept is no longer warranted
- When a node is renamed, merged, or deleted, re-point its inbound wikilinks to the node that now covers the concept, or remove them if the concept no longer exists — otherwise they linger as stale hollow links mistaken for planned concepts
- Keep target names in a consistent vocabulary so that links resolve automatically once the corresponding node is created

Description of `## Open Questions`:
- A brief (~100 words), precise summary of any gaps and conflicts within existing research on this concept
- Focus on what remains unresolved or contested in the literature
- Highlight underexplored directions and opportunities for new research
- Do not record gaps in the knowledge base's own coverage here, such as relevant articles left unread; this subsection is for questions unresolved within the literature itself
    - Coverage gaps belong in `## Synthesis`, stated where the affected claim is made

Description of `## Synthesis`:
- The primary working section of the concept node file
- The LLM determines the organization and granularity of subsections within `## Synthesis` as appropriate for the concept
- The key objectives of this subsection are as follows:
    - Summarize key findings and empirical results from relevant articles
    - Identify connections, interactions, and dependencies between ideas
    - Note methodological trends — dominant approaches and fallen-out-of-favor methods
    - Track historical evolution — foundational works versus recent developments
    - Identify the most central or influential articles for this concept
    - Note where findings generalize versus where they are narrow or setting-specific
    - Flag terminological inconsistencies across papers
    - Use in-text citations in curly brackets to attribute claims to their source articles — each citation is a key defined in the `## References` subsection (e.g., `{2020_Chen_SimCLR}`)
    - Cite every substantive empirical claim, finding, or attributed argument; chain multiple citations where a claim draws on several works (e.g., `{2020_Grill_BYOL}{2020_Chen_SimSiam}`)
    - Where the evidence for a claim is weaker than it appears — because it rests on how other works characterize a source rather than on the source itself, or because a relevant article was not consulted — say so where the claim is made, not in a separate list
    - Conversely, do not hedge claims that are adequately supported; a qualification or caveat that appears everywhere carries no information

Description of `## References`:
- Lists the `../Literature/` folders actually cited for the node
- Do not list an article that was not cited, even where it is clearly relevant to the concept
    - A work may be named in `## Synthesis` without a citation key, and the reference list should reflect only the sources the node actually cited
- Each entry consists of a citation key in curly brackets followed by the full folder name from `../Literature/`
    - Citation keys use the format `{Year_FirstAuthor}` from the folder name (e.g., `{2019_Arora}`)
    - e.g., `{2019_Arora} 2019_Arora_A_Theoretical_Analysis_of_Contrastive_Unsupervised_Representation_Learning`
- A folder has one citation key across the knowledge base — reuse the key it already carries elsewhere, and assign a new key only for a folder not yet cited
    - A folder-to-key registry in `LLM_NOTES.md` can help track these mappings
- Where two folders would produce the same key, distinguish them with a short suffix — a name from the `models` field where present, otherwise a distinctive title word
    - e.g., `{2020_Chen_SimCLR}` and `{2020_Chen_SimSiam}`
    - If a new folder collides with a key in use, distinguish the new folder, not the existing entry
- Keys are stable identifiers, not positions — never re-key an existing entry, since in-text citations depend on the key staying constant
- Because keys identify a source and stay consistent across the knowledge base, text can move between nodes on a merge or split without breaking citations — carry its `## References` entry along
- Every cited key appears exactly once in `## References`, and every `## References` entry is cited at least once
- List entries chronologically by year, then alphabetically by first author

### Math and Algorithms

Guidelines for math and algorithm presentation:
- Write inline math in LaTeX between single dollar signs (e.g., `$W^T W = F F^T$`); Obsidian renders these as typeset math
- Set a key or long equation on its own line as display LaTeX between double dollar signs (`$$ ... $$`)
- Present a multi-line derivation as display LaTeX, aligning the steps within `$$ ... $$` (e.g., an `aligned` environment)
- Present algorithms in prose or as short pseudocode in a fenced code block; Obsidian does not render LaTeX algorithm environments, so these stay plain text

Prioritize insight over transcription — consider whether a derivation or algorithm may be more concisely or more clearly presented in prose or as a pointer to the source article.
Note that citation keys sit in prose and LaTeX braces sit inside `$...$` or `$$...$$`, so the `{...}` used by each never collides.

### Template

```
---
title: "Human-Readable Concept Name"
tags: ["tag1", "tag2"]
last_updated: "YYYY/MM/DD"
---

## Summary
Freeform text description.

## Related Concepts
[[related_concept_1]]
[[related_concept_2]]

## Open Questions
Freeform text description.

## Synthesis
Freeform text description.

## References
{reference_key_1} folder_name_in_Literature_1
{reference_key_2} folder_name_in_Literature_2

```
