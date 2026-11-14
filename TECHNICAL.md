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
tap-to-filter work on touch screens; tapping a city scrolls to the filtered list. Dots keep a
minimum size on screen at every zoom (4.5 px radius, 6 px on touch screens) and a tap or click that
misses selects the nearest dot within reach (24 px for a finger, 9 px for a mouse). The year
slider's handles have a grab area as tall as the slider (36 px; 52 px on touch or narrow screens)
and a tap on the track moves the nearer handle.

Caveats: 207 entries are low confidence and 208 are flagged borderline; a mention records that a
text names a fair, not how much it says.

## 4. Study pages (`fairs/*.html`)

Scope: the 59 fairs dated 1800-1970 with ten or more citing texts, (section 4.9) 88 imperial and colonial
exhibitions with two or more citing texts, and (section 4.8) 71 exhibitions
in East and Southeast Asia. All files are in `fp/`.

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

### 4.8 Pages for exhibitions in East and Southeast Asia
Scope: every exhibition in the timeline held or planned in East or Southeast Asia with at least
one citing text, 71 beyond Osaka 1970 (51 held, 9 never held, 7 cancelled, 4 planned), including
events after 1990 that the timeline page hides. `mk_asia_keys.py` writes `keys_asia.py` (slug,
years, place pattern, hand-picked name pattern); `keys.py` merges it and exposes `ASIA_IDS`.
For these fairs `dossier.py` also takes the lines the timeline sweep recorded as hits, so a fair
named without its year is still found. `asia_pack.py` writes `asia_in/<id>.txt` (chapters that
treat or mention the fair). One agent per fair (brief `ASIAFAIR_BRIEF.md`) read the dossier and
the chapters, wrote `notes/<id>_p1.jsonl` and `essay/<id>.md`; length follows the sources (164 to
1,743 words; about 38,800 in all; 41 pages rest on one text). The `# ASIA` section on these pages
means participants and visitors from other parts of Asia and from colonies. Web research
(`WEB_BRIEF.md`, 21 agents, several fairs per agent for the minor ones) covered about 45 fairs:
after verification 119 primary-source links on 32 pages and 74 web resources on 29 pages; for
many small fairs nothing reliable is online. `wiki_asia.py` resolved 21 English Wikipedia
articles in two batched requests (the series article serves the five Japanese National
Industrial Exhibitions). `ASIA=1 python3 deepcheck.py` → `asia_check.json`: 2,256 citations, all
to existing lines; 138 of 141 quotations verbatim (three printed without quotation marks); 82
citations (4%) without a nearby name or number. Agents used: 92 of 200.

`build.py` shows a status label for fairs never held, cancelled or only planned, and
`fairs/index.html` lists the pages in two groups.

Things found on the way:
- **Text 58 line numbers.** `cand/58_full.txt` contained 320 form feeds, which some tools count
  as line breaks, and its first dossiers were cut from an older extraction. The form feeds were
  removed (numbering by newline unchanged; page starts kept in `fp/58_pagestarts.json`);
  `fix58.py` tested all 443 citations to text 58 and moved four.
- **Timeline attribution.** Six mentions from text 08 (Denton) had been filed under text 09
  (`tl/out/chunk_09.jsonl` is text 08's chunk). Patched in `tl/clusters.json` and
  `tl/entries_final.json` (affects Nanjing 1910, Hangzhou 1929 and 2002, Shanghai 2010, New York
  1939 and 1964); re-running `tl/final.py` would undo the patch in `entries_final.json`.
- **Timeline records the texts do not support** (not corrected; the pages say what the texts
  say): the 1907 "Sixth Regular Domestic Exhibition" recorded as never held may be the Tokyo
  Industrial Exposition of 1907, which was held; Tokyo 1912 "Colonial Exposition" and "Grand
  Japanese Exhibition" may overlap; Tianjin, Wuhan and Chengdu 1907 are one sentence about
  local-products shows "since 1907"; Beijing 1999 and Kunming 1999 are one event that moved;
  several "never held" or "cancelled" labels are inferences; visitor figures for Tokyo 1890 and
  the cancelled Tokyo 1996 belong to other fairs.
- The Osaka 2025 essay misread an attendance figure for the earlier Osaka expo as 2025's and
  was corrected by hand; other pages were not read by a person.

### 4.9 Pages for imperial and colonial exhibitions

Added 14 November 2026: 88 pages for fairs that had no page, are cited by two or more texts, and have an
imperial or colonial side. The list (`tmp/colonial_nopage.json`, made by a one-off script) took every such fair
that the timeline classes as colonial, that was held in a colony before 1960 or in Australia, New Zealand or
Canada before 1931, or whose name or description uses words such as colonial, empire, imperial or India; six
false hits were removed by hand. For about thirty of the 88 the connection is slight (a colonial section, or a
word in the timeline description), and for a few the texts turned out to show none: those pages say so.

- Keys: `fp/mk_emp_keys.py` → `fp/keys_emp.py` (`KE`), then edited by hand (over-broad name patterns removed, the
  monograph on London 1910 marked), so do not regenerate it. `fp/keys.py` exposes `EMP_IDS` and
  `SMALL_IDS` (Asia + imperial pages).
- One haiku agent per fair (88): brief `fp/EMP_BRIEF.md`, the Asia brief with a compulsory section "Empire and
  colony". Input: `fp/dossier/<id>/part_1.txt` and `fp/asia_in/<id>.txt` (`fp/asia_pack.py <ids>`). Output:
  `fp/notes/<id>_p1.jsonl`, `fp/essay/<id>.md`.
- Check (`ASIA=1 python3 fp/deepcheck.py <ids>`, results merged into `fp/asia_check.json`): about 60,600 words,
  3,369 line citations, none pointing at a missing line or unknown text; 17 of 204 quotations did not match
  the source and are shown without quotation marks.
- Web and primary sources: 22 haiku agents, four fairs each (`fp/EMPWEB_BRIEF.md`, `fp/empweb/group_KK.txt`);
  after `fp/verify.py`, 128 web resources and 166 primary sources on 75 pages; 13 pages have none.
- Wikipedia introductions for 38 pages (`fp/wiki_emp.py`, three batched requests; titles taken from the saved
  "List of world's fairs").
- `fp/build.py` lists them in a third group on `fairs/index.html`.

Known weaknesses:

- **Text 33 (Maxwell, *Colonial Photography and Exhibitions*) was not used.** It is a scan without a text
  layer, so the agents could not read it, although the timeline cites it for several of these fairs.
- Eleven pages are under 300 words because the texts only list the fair (usually in the "Fairs Not Included"
  appendix of the Findling and Pelle reference works).
- The agents were told to use nothing but the texts, so where the texts do not say that a place was a colony
  the "Empire and colony" section says the texts show no connection, even for a fair held in a colony. The
  facts box on the page still gives the country as the timeline has it.
- The agents reported many points where the texts do not support the timeline record (scope labels above
  all, also dates, visitor figures and names). These reports are in the session transcript only; the timeline
  has not been corrected.
- The 22 web agents ran up to 20 at a time, each with up to 12 Internet Archive searches. No request was
  refused, but this was parallel use of archive.org and should be staggered next time.

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

## 4b. Approaches page (`approaches.html`)

All files are in `ap/`. Agents: 54 readers + 13 writers (taken from the unspent part of the
200 allowed for the Asia pages).

1. **Reading.** One agent per text (brief `APPROACH_BRIEF.md`; the two picture books skipped)
   read the contents overview (`toc/bookinfo/NN.txt`) and then the introduction, conclusion and
   theoretical passages of the full text, and wrote `appr/NN.json`: `stance`, `concepts` (name,
   kind, author, definition, draws_on, used_for, fairs, one exact quotation with line, lines, and
   `carries_over`, the agent's own suggestion), `against`, `methods`, `gaps`. Most agents read
   1,000-2,500 lines, not whole books, and say so in `note`.
2. **Grouping and checking.** `themes.py` assigns each of the 452 concepts to one of 13 themes
   by a regular expression on its name (first match wins; the list is in the script), checks
   every quotation against the text (all 452 occur verbatim) and every line number, and writes
   `concepts.json`, `books.json` and `theme_in/<key>.json`.
3. **Writing.** One agent per theme (brief `THEME_BRIEF.md`) wrote `theme/<key>.md`: INTRO,
   STRANDS, DEBATES, CARRYING IT OVER, SEE, with citations `[NN, l. LINE]`. They worked from the
   records and opened the texts where unsure; several say which lines they did not open.
4. **Building.** `build_approaches.py <project dir>` writes the page: essays with page-linked
   citations (to `contents.html#tNN`) and fair names linked to study pages; all concept cards per
   theme (collapsible, searchable); theorists named as sources by two or more texts; the 30
   most-cited works in the bibliography's theory section; each text's stance, opponents, methods
   and gaps; and a list of all gaps the authors name. A quotation in an essay is kept in
   quotation marks only if it occurs in a text cited in the same paragraph (85 of 88).

Limits: a concept's theme is decided by its name, so some sit oddly; definitions report the
author's use in that text, not the theorist's; "carrying it over" and "possible use elsewhere"
are suggestions; the writers flagged a few records as doubtful (wrong fair attached to a
concept, reversed line pairs, an author field naming the theorist instead of the author), of
which only one was corrected.

## 4c. Third pass for primary sources, 1880-1950

- `ps/ndl_search.py`: title searches of the National Diet Library (NDL Search OpenSearch API,
  restricted to NDL Digital Collections and to a window of years round the fair) for the
  Japanese names of about 65 fairs, including Western fairs in which Japan took part; a
  candidate is kept only if its IIIF manifest exists, which is the case only for items open to
  the public internet. Sequential, cached in `ps/ndl/`, stops on any error; the API answered 429
  after about forty requests at 3-5 second spacing, so it was resumed at 16 and 8 seconds.
  `ps/ndl_merge.py` adds the finds to `fp/web/<id>.json` (`"added": "third pass (NDL)"`). The
  Japanese names in the script are search terms supplied editorially.
- modernhistory.org.cn (近代史资料数据库): title searches through its search endpoint for the
  Chinese exhibitions and for fairs in which China took part; results in `ps/mh/`. Eighteen
  records chosen by hand from the titles were added to seven pages (Nanjing 1910, Hangzhou 1929,
  San Francisco 1915, Philadelphia 1926, Tokyo 1914, and reports of Wuhan and Chengdu fairs whose
  records give no date). Page addresses use the route the site itself builds
  (`#/DocumentDetails_ts_da?fileCode=…` for books, `DocumentDetails_qk` for periodicals); the site
  is a single-page application, so a link check cannot tell a good record address from a bad one.
- Neither set was read: descriptions rest on catalogue titles.

## 4d. Exhibitions not in the texts (`uc/`, timeline "Show" control)

Added 14 November 2026. The timeline shows by default only fairs that at least one text
mentions. A second set, kept apart in the data (ids `W0001`…, empty list of texts), holds
exhibitions of 1850–1950 known from Wikipedia or other online sources. The "Show" control on
the timeline offers "cited in our texts" (default), "all, including not cited" and "not cited
in our texts". Uncited entries have a dotted left edge and a "not in our texts" tag, list their
online sources in place of the texts, and places with only uncited fairs are drawn as pale
dashed dots. They never count towards the figures in the page header.

How the first list (306 entries) was made:

1. `uc/cities.json`: a hand-made list of 258 larger cities of the period; the 202 with two or
   fewer held exhibitions 1850–1950 in the timeline were searched.
2. `uc/wikilist.json`: Wikipedia's English "List of world's fairs", fetched once and parsed
   (436 entries for 1850–1950; about 160 had no counterpart in the timeline). The Japanese
   article 博覧会 supplied a list of expositions in Japan.
3. 19 haiku agents (`uc/UC_BRIEF.md`, input `uc/in/batch_KK.txt`, output `uc/out/`), about 11
   cities each, searched the web in English and local languages, were told to check the
   Wikipedia entries ("seeds") against another source, to return only distinct exhibitions with
   a source they had seen, and not to contact Wikipedia themselves. They returned 211 events.
4. `uc/merge.py` drops events outside 1850–1950, events the timeline already has (same city
   under any of its names, year within one, similar name), and duplicates; it adds the
   Wikipedia-list entries no agent returned. `uc/checklinks.py` requested each of the 256 source
   links once: 218 answered 200, 2 were dead and were removed, the rest refused a script or
   timed out and were kept. `uc/finalize.py` confirms Wikipedia article titles and looks up
   coordinates in batches, assigns regions and ids, and writes `uc/uncited.json`, which
   `tl/export.py` adds to `timeline.html` and writes out as `timeline_not_in_texts.csv`.

What to know about its quality:

- 132 entries were found by web search alone, 63 by web search and also in a Wikipedia list, and
  111 rest on a Wikipedia list only (column `found_by`); the last group has confidence "low".
- Agents often could not open a page and worked from the search engine's summary of it; the
  `notes` column says so entry by entry. Dates and visitor figures are the weakest fields.
- For recurring fairs there is one entry, for the founding year.
- The agents found nothing usable for 74 of the 202 cities, among them Beijing, Tianjin,
  Guangzhou, Wuhan, Istanbul, Warsaw, Algiers, Lima and St. Louis. That reflects a short search,
  not an absence of exhibitions.
- Matching against the timeline was by city and year; an event the texts place in a different
  city or year may appear twice.
- Wikipedia traffic for the whole task was 9 requests. The rules for any future Wikimedia
  lookups are in `pipeline/WIKIPEDIA.md` and enforced in `fp/wiki.py` (cache, 8 s spacing,
  15 requests an hour, 40 a day, stop for an hour after any refusal).

Hand additions go in `uc/out/batch_9x.jsonl` (any city; `batch_90` holds Stockholm 1851 and 1909, found from a
bibliography title).

**Fairs the first extraction missed (`tl/extra_cited.json`).** On 14 November 2026 the bibliography was
screened for titles naming an exhibition that neither list had (`bibhints/screen.py`, `screen2.py`; 936 titles
read by hand). Five turned out to be discussed in the texts and were added to the cited timeline by hand, with
text and line references, under ids `X001`–`X005` (Mladá Boleslav 1912, the eugenics exhibitions of 1921 and
1932 in New York, Proudhon's perpetual-exhibition project, Orvieto 1896). They live in their own file, which
`tl/export.py` appends, so that the `F####` ids the study pages depend on never shift. The cited timeline
therefore has 850 entries (804 up to 1990).

To add more: put further `batch_KK.jsonl` files in `uc/out/` (same format), then run
`uc/merge.py`, `uc/checklinks.py`, `uc/finalize.py` and `tl/export.py <site folder>`.

## 5. Publishing

The project folder is a git repository (`main`, remote `origin` on GitHub, Pages served from
the branch root). `.gitignore` excludes the books (all in `books/` since 14 November 2026; before that in the
project folder and `second-wave/`) and `pipeline/`. `pipeline/`
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
