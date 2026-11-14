# Technical description

This file records how the bibliography, timeline and study pages were built, where every piece
lives, how to rebuild, and what is known to be weak. `README.md` is the short version.

Everything was produced in one long assistant session in November 2026. The working files were
in a temporary scratch folder; they have been copied to `pipeline/` (not in git). **Paths below
are relative to `pipeline/` unless they start with a project file name.** Scripts find their own
folder, so they run from `pipeline/` as they ran in the scratch folder.

## 0. Rules the work followed

- Facts come from the texts only. Agents were told never to add dates, names or figures from
  memory; conflicts between texts are reported, not settled.
- Reading was delegated to small-model ("haiku") sub-agents, at most 20 at once. Budgets set by
  the owner: 100 agents for the timeline (91 used), 200 for the study pages (188 used),
  215 for the contents page (217 used, the two extra taken from the study-page remainder),
  6 for a second pass on primary sources, 59 for deepening the essays, 200 for the East and
  Southeast Asia pages.
- Wikipedia/Wikidata: few, batched, cached, sequential requests from one script, with a
  non-identifying user-agent. Internet Archive: searched only with the `ia` command-line tool,
  slowly, with caps per agent. No personal details in any request.
- Git: nothing is committed or pushed without the owner's explicit say-so each time.
- The scanned PDF without a text layer (text 33, Maxwell) was read by eye from page images, for
  its bibliography only.

## 1. The texts

`manifest.json` (first wave, texts 01-29 and 58) and `manifest2.json` (second wave, 31-57) list
each file with its number. `tl/sources.json` maps number → short label; `fp/cites.py` maps number
→ full citation and author-date label (citations supplied editorially, not extracted).

Text extraction (NOT saved in `pipeline/`, because it is the books' text):
- `txt/NN.txt`, `txt2/NN.txt`: `pdftotext -layout` output per PDF, form feeds kept; epubs
  converted to plain text.
- `cand.py` reads those and writes `cand/NN_full.txt` (same text, form feeds replaced by
  newlines, very long epub paragraphs wrapped at 300 characters) and `cand/NN.txt` (passages
  mentioning an exhibition near a year). **Every line number quoted anywhere in the data refers
  to `cand/NN_full.txt`.** To rebuild, recreate `txt/` and `txt2/` with the same numbering, then
  run `cand.py`.

Texts with nothing to extract: 10 (Brown), 51 (Debord), the two Cotter picture books for
bibliography purposes. Text 33 has no text layer.

## 2. Bibliography (`bibliography.csv`, `bibliography.html`)

1. Agents (briefs `BRIEF.md`, `BRIEF2.md`) transcribed each text's bibliography to
   `out/*.jsonl` and `out2/*.jsonl`, one work per line (section, authors, year, title,
   translation, type, container, pages, place, publisher, chapters citing, note).
2. `merge.py <outdir>` loads both folders, merges duplicates with a union-find on first-author
   surname + main title (fuzzy, with guards for digits and roman numerals, plus a cross-surname
   pass), applies manual overrides in `force.json`, and writes `bibliography.csv`, `final.json`
   and `near.txt` (near-misses for review). The two Findling & Pelle editions count as one citing
   text (`SRCMAP`). `diag.py` lists merged clusters whose titles differ.
3. `build_html.py <outdir>` + `template.html` produce the page: section tabs, filters, sortable
   columns, citation bars, single-row selection that exposes one COinS record for Zotero, a blue
   dot for works held in the folder (`held.json`).

Result: 9,431 works (3,995 secondary, 1,029 theory, 4,407 primary). Where a text has a
bibliography, its notes were not mined. Extraction was not verified entry by entry.

## 3. Timeline (`timeline.csv`, `timeline_mentions.csv`, `timeline.html`)

1. `chunk.py` cut the `cand/NN.txt` passage files into 54 chunks (`tl/chunks/`, not saved).
   Agents (brief `tl/TL_BRIEF.md`) listed every fair in a chunk → `tl/out/*.jsonl` (also
   appendix tables of the two reference works, the scan, and extra Asia-focused passes).
2. `tl/cluster.py` grouped mentions by city and year → `tl/clusters.json` (1,035 clusters).
   `tl/batches.py` made 23 batches; consolidation agents (brief `tl/SYN_BRIEF.md`) wrote one
   clean entry per fair → `tl/syn_out/`.
3. `tl/assemble.py` → `entries_raw.json`; `tl/final.py` applies manual merges, corrections and
   editorial notes → `tl/entries_final.json` (845 fairs: 721 held, 67 never held, 33 cancelled,
   24 planned; 4,402 mentions).
4. `tl/mapbuild.py` supplies coordinates (hand-entered, then replaced by Wikipedia's from
   `tl/wikicoords.json`, all within 40 km) and draws the land outline from
   `mapdata/countries-110m.json`.
5. `tl/export.py <project dir>` + `tl/tl_template.html` write the three timeline files. It also
   reads `fp/done.json` so cards link to study pages.

Page behaviour worth knowing: fairs opening after 1990 are in the data but hidden (`CUT` in the
template); map dots use five fixed classes (1, 2-4, 5-9, 10-19, 20+ fairs); pinch zoom and
tap-to-filter work on touch screens; tapping a city scrolls to the filtered list.

Caveats: 207 entries are low confidence and 208 are flagged borderline; a mention records that a
text names a fair, not how much it says.

## 4. Study pages (`fairs/*.html`)

Scope: the 59 fairs dated 1800-1970 with ten or more citing texts. All files are in `fp/`.

### 4.1 Dossiers (not saved)
`fp/keys.py` holds, per fair, a slug, the years, a context pattern, a "strong" name pattern and
the texts wholly about that fair. `fp/dossier.py` scans every text for lines matching the fair
(strong pattern, or a year with the context pattern within two lines; publisher-and-date lines
excluded), takes about 650 characters either side, drops index pages, keeps the densest passages
up to a cap per text (13 KB; 30 KB for reference works 35, 39, 50; 42 KB for books about the
fair), and writes `dossier/<id>/part_K.txt` in parts of about 190 KB with each line's number
and each passage's page header. `fp/dossier_index.json` lists the parts. `fp/asia.py` builds
`dossier/<id>/asia.txt`: passages naming East, South or Southeast Asia within 14 lines of a
mention of the fair. Rebuild both after recreating `cand/`.

### 4.2 Agents and their outputs
| Brief | Output | What |
|---|---|---|
| `READ_BRIEF.md` | `notes/<id>_pK.jsonl` | Per text: depth (central … passing, or none), where in the text, a publishable annotation, up to 12 claims with line numbers, up to 2 exact quotes with line numbers, further reading recommended by reference works, theme tags. |
| `WRITE_BRIEF.md` | `essay/<id>.md` | `# LEAD`, `# ESSAY` (sections, last one "How historians have interpreted it"), `# ASIA`, `# THEMES`, `# QUESTIONS`. Citations as `[NN, l. LINE]`. |
| `ASIA_BRIEF.md` | `asia/<id>.md` | The Asian-presence section for the 11 essays written before that section was added to the write brief. |
| `WEB_BRIEF.md` | `web/<id>.json` | Web resources and open-access primary sources. Every URL had to be one the agent opened or received from a search. |

Fairs with one dossier part had one agent do read then write; larger fairs had one reader per
part and a separate writer. Calcutta 1883 and Osaka 1970 are in Asia and have no `asia.txt`.

The first 11 essays (F0380, F0100, F0067, F0162, F0132, F0182, F0205, F0060, F0077, F0215,
F0222) cite text numbers without line numbers, so their in-text references show no page.

### 4.3 Checks done by script
- **Quotes** (`build.py`, `quote_ok`): shown only if the quote, stripped to letters and digits,
  occurs in the text's `cand/NN_full.txt`.
- **Page numbers** (`pages.py`): a line number becomes a printed page by finding the page it
  falls on and reading printed numbers from page headers and footers (handles `[ 48 ]` style and
  two-page spreads). No page is shown for epubs (02, 52-57) or text 38.
- **Links** (`verify.py` → `web_ok/<id>.json`): Internet Archive identifiers are looked up in one
  `ia search` per fair and missing ones dropped, with title, creator, year and language taken
  from the archive's record; every other URL is requested once and kept on 200/202/403/405/406/429
  (many institutions refuse automated requests), dropped otherwise. Results are cached.
- **Wikipedia** (`wiki.py` → `wiki.json`): article, opening extract, lead image (shown only when
  hosted on Commons), host-language article and Commons category, in eight cached requests.

### 4.4 Second pass for primary sources
`fp/ia2.py` runs a few targeted `ia search` calls for each thin page, one at a time with pauses
(cache in `fp/ia2/`), and prints candidates not already listed. Relevant items were chosen by
hand from titles and appended to `web/<id>.json` with `"added": "second pass"`; their one-line
descriptions rest on the catalogue title, not on reading the item. Its first round filtered on
the `date` field, which many older items lack, so it under-finds; a round using title searches
without a date filter is the obvious next step. See section 6 for what is still thin.

### 4.5 Building
`fp/build.py <project dir> [ids]` writes `fairs/<slug>.html`, `fairs/index.html`,
`fairs/fair.css` (from `fp/fair.css`) and `fp/done.json`. It builds a page only when an essay and
notes exist. "Further reading" comes from `final.json` by matching the fair's patterns against
titles. `fp/rebuild.sh` runs verify, build and the timeline export in one go (it contains an
absolute path to the original scratch folder; edit it or run the three steps by hand).
`fp/sync.sh` is the command that copied the scratch folder to `pipeline/`.

`fp/STATE.md` is the running log kept during the work.

### 4.6 Chapter guide and "about the work" (from the contents work, section 4a)
`fp/chapters.py` joins the study pages to the chapter overviews in `toc/`. For each fair,
`guide()` returns: every chapter of a book wholly about the fair (the `mono` list in `keys.py`);
chapters whose overview agent listed the fair among those "discussed at length" (city and year
match); and chapters whose overview text names the fair (strong pattern, or year plus context
pattern). `build.py` shows these as the "Chapter guide" section, each chapter collapsible with
its overview, printed page and a link to `contents.html#tNN-IDX`. Chapters whose overview
mentions an Asian country or people carry an "Asia" tag and are listed at the end of the
Asian-presence section. Each annotated-bibliography entry gains a collapsible "About the work
as a whole" (book overview, Asia note and level from `toc/book/NN.json`). No agents were used
for this step; the overviews describe whole chapters, not only what they say about the fair,
and the third group is a keyword match that can include chapters mentioning the fair in passing.
Two pages have no guide (no chapter matched).

### 4.7 Deepening the essays
One agent per fair (59; brief `DEEP_BRIEF.md`) was given `deep_in/<id>.txt` (from `deep_pack.py`:
the chapters that treat the fair, with file and line range and overview, followed by the essay as
it stood) and told to read the most relevant chapters in the full text and write ADDITIONS, not a
rewrite: `deep/<id>.md` with `# MORE` (new sections, or paragraphs under an existing section
heading copied exactly), optional `# ASIA`, and `# QUESTIONS`. `build.py` merges them: a heading
equal to an existing one appends its paragraphs to that section; a new heading becomes a section
placed before "How historians have interpreted it". Citations in the additions may name any text
(a text without an annotation on the page links to the contents page instead).

Result: about 56,000 added words and 2,727 citations (essays now about 140,000 words in all).
`deepcheck.py` (→ `deep_check.json`) tests every citation: all point to existing lines; 163 of 164
quotations occur verbatim in a cited text, and the one that does not (line broken by a column
fault in the extraction) is printed without quotation marks; for 102 citations (4%) no name or
number from the sentence occurs within about 50 lines of the cited line, mostly sentences that
summarise an argument. Agents read less than asked (typically 1,000-3,000 lines) where the texts
say little about a fair.

**What the agents found wrong in the earlier essays (not corrected; the additions state the
disagreement in the text):** figures that differ between texts (e.g. 245 against 254 competition
designs for 1851; cholera deaths and site size at Vienna 1873; Trylon height and Democracity
population at New York 1939; Soviet budget and attendance at Brussels 1958); statements the
cited text does not support (who built the Paris 1855 fine arts palace; Shantyville placed at
New York 1853 instead of Philadelphia 1876; Douglass at the 1876 opening; the labour agreement at
San Francisco 1915; who organised Negro Day at Chicago 1933); and a number of citations whose
line number is a few lines off or names the wrong text (Antwerp 1930 "most lavish display"
belongs to text 39, not 35; several in Paris 1937 and Brussels 1958). The agents' final reports
with the full lists are in the session transcript, not in `pipeline/`.

## 4a. Contents page (`contents.html`)

All files are in `toc/`. Agents used: 217 (budget 215 + 12 left from the study pages).

1. **Finding the chapters.** `outlines.json` holds PDF bookmarks where a file has them;
   `seg.py` turns them into line hints (`seg/NN.json`). One agent per text (brief `MAP_BRIEF.md`)
   read the printed contents page and the text and wrote `map/NN.json`: one unit per part,
   chapter, front- or back-matter section, with kind, number, title, author, level, printed page
   and first and last line in `cand/NN_full.txt`. Text 33 (scan) has `map_notext/33.json`, its
   contents page read by eye, and no overviews.
2. **Chapter overviews.** `batch.py 260` packed the 606 units into 118 batches
   (`batch/batch_KKK.json`, about 26 MB of text in all). One agent per batch (brief
   `SUM_BRIEF.md`) read every unit in full and wrote `sum/batch_KKK.jsonl`: text, idx, overview
   (110-190 words for a chapter, 40-80 for front matter or a short unit), fairs discussed at
   length, and a note on boundary problems. The long "Entries" blocks of the two reference works
   (35 idx 2, 39 idx 3) were not batched; `hand.json` gives them a fixed sentence plus the entry
   headings picked out of the text by a regular expression (101 and 72 headings).
3. **Book overview and Asia note.** `bookinfo.py` writes `bookinfo/NN.txt` (contents plus the
   chapter overviews); `bookasia.py` writes `asia/NN.txt` (lines per chapter that name East or
   Southeast Asia, and the densest passages). One agent per book, small ones in pairs (brief
   `BOOK_BRIEF.md`), wrote `book/NN.json`: `overview`, `asia` (one or two paragraphs) and
   `asia_level` (central, substantial, occasional, slight). **These rest on the chapter overviews
   and the keyword-found passages, not on a fresh reading of the book**; most agents said they
   did not open the full text beyond the passages. A level of "central" can mean a book about an
   exhibition held in Asia (08, 17, 28, 58) as well as one about Asian participation abroad.
4. **Building.** `build_toc.py <project dir>` writes `contents.html`: search, a filter on the
   Asia level, each book collapsible with overview, Asia note and contents; fair tags link to a
   study page when city and year match one. Printed pages come from the map or from `pages.py`.

Known problems: some unit boundaries are a few lines off (noted in the `note` field of the
`sum` files); authors of the two appendices of text 39 looked swapped in the map and are not
shown; overviews of chapters without a named author use the book's author; picture-book units
(48, 56) are mostly captions.

## 5. Publishing

The project folder is a git repository (`main`, remote `origin` on GitHub, Pages served from
the branch root). `.gitignore` excludes the books, `second-wave/` and `pipeline/`. `pipeline/`
is excluded because its briefs contain local paths and its notes hold extended material derived
from the books; publish it only deliberately.

## 6. Known weaknesses and open work

- Essays, annotations and timeline entries were not checked against the sources by a person.
  Agents reported several places where the earlier timeline record disagreed with the text they
  read (acreages, attendance, a few names); those timeline values have not been corrected.
- Appendix statistics tables in texts 35 and 39 are rotated in the PDFs and came out garbled;
  agents were told to leave unreadable figures out.
- Texts 09 and 58: a few line numbers in older timeline mentions do not match the current
  `cand` files.
- Primary sources are thin (fewer than about 8 items) for some pages, mainly where material sits
  in libraries that refuse automated requests (HathiTrust, Gallica, Trove, Library of Congress):
  at the time of writing Melbourne 1880, Glasgow 1888 and 1938, Barcelona 1888 and 1929, Liège
  1930, Antwerp 1930, Paris 1931 and 1937. Count the items in `fp/web_ok/` for the current state.
- Web-resource links marked `"checked": "search result only"` were never opened by the agent.
- The Asian-presence section depends on keyword-found passages; a fair's Asian displays described
  without a nearby mention of the fair's name or year can be missed (Expo 67 is the clearest case).
- The owner counted 60 qualifying fairs; the data gives 59.
- Not done: a systematic second sweep of non-Asia texts for fairs the passage filter missed.
