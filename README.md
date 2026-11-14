# Exhibitions and World's Fairs: a research aid

Built from about 55 scholarly books and articles on the history of exhibitions (the PDFs and
epubs in this folder and in `second-wave/`, which are not in the repository).

**For how any of this was made, what the files contain, how to rebuild it, and what is known to
be unreliable, read [`TECHNICAL.md`](TECHNICAL.md).** Anyone continuing the work, including a
future assistant session, should start there.

## What is here

| File | What it is |
|---|---|
| `index.html` | Landing page linking the five tools below. |
| `timeline.html` | 804 exhibitions up to 1990 (850 in the data) with a world map, year slider and filters. Cards for the most-discussed fairs link to their study page. |
| `timeline_not_in_texts.csv` | 306 exhibitions of 1850–1950 that none of the texts mention, found in Wikipedia lists and by web search for large cities the timeline covered thinly. Hidden on the timeline unless "Show" is set to "all" or "not cited". A first list, not checked one by one. |
| `timeline.csv`, `timeline_mentions.csv` | The timeline data: one row per fair, and one row per mention of a fair in a text (with line numbers). |
| `bibliography.html`, `bibliography.csv` | 9,431 works cited across the texts, merged and deduplicated, with citation counts and Zotero import. |
| `fairs/` | 218 study pages: the 59 fairs between 1800 and 1970 that ten or more texts discuss, 71 exhibitions held or planned in East and Southeast Asia that at least one text mentions, and 88 exhibitions with an imperial or colonial side that two or more texts mention (many of the last two groups are short). Each has: long description, Asian presence, themes, study questions, a chapter guide (which chapters of which books treat the fair, each with an overview), annotated bibliography, further reading, primary sources online, web resources. `fairs/index.html` lists them. |
| `contents.html` | The table of contents of every text, with a machine-written overview of each chapter (608), a short overview of each work, and a note on how far it discusses East and Southeast Asia (filterable by level of coverage). |
| `approaches.html` | The theoretical framings, concepts and methods used in the texts: thirteen thematic essays with "carrying it over" suggestions, 452 concept records (definition, quotation, page), theorists drawn on, each text's framing, opponents, methods and stated gaps. |
| `pipeline/` | Scripts, agent briefs and intermediate results used to build everything. Kept out of git (see `.gitignore`). |

Published at <https://kmlawson.github.io/exhibitions/> from the `main` branch.

## Three things to know before relying on it

1. **Machine reading, not checked line by line.** Small-model agents read the texts and wrote
   the notes, essays and annotations. Facts come only from the texts; a blank means no text gave
   the value. Verify against the source before citing.
2. **Quotations and links were checked by script**: a quotation appears on a page only if it
   matches the source text exactly; every Internet Archive identifier was confirmed to exist and
   every other link was requested once.
3. **Rebuilding needs the books.** The scripts in `pipeline/` regenerate the pages from saved
   intermediate results, but the text extractions of the books are not saved and must be
   re-created from the PDFs first. `TECHNICAL.md` says how.
