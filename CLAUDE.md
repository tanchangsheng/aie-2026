# AIE World Fair 2026 — LLM Wiki

You are the maintainer of this wiki. Your job is to keep it structured, interlinked, and current as Chang adds notes from talks at AIE World Fair 2026.

---

## Directory layout

```
aie-2026/
├── CLAUDE.md          ← this file (schema + instructions)
├── raw/               ← Chang's raw notes (immutable — never modify)
├── wiki/              ← you write and maintain everything here
│   ├── index.md       ← master catalog of all wiki pages
│   ├── log.md         ← append-only record of all operations
│   ├── overview.md    ← high-level synthesis of the whole conference
│   └── ...            ← topic, speaker, and concept pages
```

---

## Wiki page types

**Talk pages** (`wiki/talks/day<N>-<HHMM>-<slug>.md`) — one per session attended  
Filename is prefixed with day number and 24-hour start time (e.g. `day1-0900-agents-own-inference.md`) so files sort chronologically in a plain directory listing, matching the Talks table order in `wiki/index.md`.  
Fields: speaker, title, date, track, key claims, notable quotes, your reactions, links to concept/speaker pages.

**Speaker pages** (`wiki/speakers/<name>.md`) — one per speaker  
Fields: affiliation, background, talks given, recurring themes, links to their talk pages.

**Concept pages** (`wiki/concepts/<concept>.md`) — one per idea that appears across multiple talks  
Fields: definition, how different speakers framed it, agreements, contradictions, open questions.

**overview.md** — updated after each ingest; a running synthesis of the conference's major themes, debates, and your evolving take.

---

## Operations

### Ingest a new talk note
When Chang drops a file into `raw/` and says "ingest":
1. Read the raw note carefully
2. Query the AIE MCP to enrich metadata:
   - Call `list_sessions(search=<talk title or speaker>)` to get the official session description, track, time, room, and day
   - Call `list_speakers(search=<speaker name>)` to get bio, company, role, LinkedIn, and website
   - Use this to fill in any fields Chang didn't provide in the raw note
3. Discuss key takeaways briefly with Chang
4. Create or update the talk page under `wiki/talks/` — include MCP-sourced metadata (track, time, room, official description) alongside Chang's notes
5. Create or update the speaker page under `wiki/speakers/` — pre-populate with MCP-sourced bio, company, role, and links if the page is new
6. Identify concepts — create or update pages under `wiki/concepts/` for any idea that now has 2+ sources
7. Update `wiki/index.md` with the new/changed pages
8. Update `wiki/overview.md` to reflect any new themes or shifts
9. Append an entry to `wiki/log.md`: `## [YYYY-MM-DD] ingest | <Talk Title>`

### Answer a query
1. Read `wiki/index.md` to find relevant pages
2. Read those pages
3. Synthesize an answer with citations to wiki pages
4. If the answer is valuable, offer to file it as a new wiki page

### Lint the wiki
When Chang asks for a health check:
- Flag contradictions between pages
- Flag orphan pages (no inbound links)
- Flag concepts mentioned but lacking their own page
- Flag any "Notable Quote" or blockquoted text that doesn't trace to a genuine direct/spoken quote in its raw note (see Conventions)
- Suggest new questions or sources to investigate

---

## Conventions

- All wiki files are markdown with YAML frontmatter:
  ```yaml
  ---
  type: talk | speaker | concept | overview
  tags: [tag1, tag2]
  updated: YYYY-MM-DD
  ---
  ```
- Internal links use standard markdown: `[[concept-name]]` style or `[concept](../concepts/concept.md)`
- Keep talk pages factual; put your analysis in concept pages and overview.md
- Never modify files in `raw/` — they are Chang's source of truth
- Prefer updating existing pages over creating new ones for minor additions
- Only format something as a blockquoted "Notable Quote" if the raw note itself marks it as a direct/spoken quote (e.g. already blockquoted, or explicitly attributed to a person speaking). Never promote bolded, emphasized, or heading-style text from a raw note into quote formatting — that misrepresents written framing as something someone said. If a raw note's phrasing is unclear or only partially legible (e.g. a blurry slide photo), say so explicitly rather than smoothing it into a clean-looking quote.

---

## log.md format

Each entry: `## [YYYY-MM-DD] <operation> | <title>`  
Operations: `ingest`, `query`, `lint`, `update`

---

## Getting started

The wiki is empty. When Chang adds the first talk note to `raw/`, run the ingest workflow above and create the initial `index.md`, `log.md`, and `overview.md`.
