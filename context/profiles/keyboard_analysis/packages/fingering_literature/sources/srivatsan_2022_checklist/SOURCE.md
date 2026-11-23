# Checklist Models for Improved Output Fluency in Piano Fingering Prediction

Source record for the `fingering_literature` package of the `keyboard_analysis` profile (see `source.yaml` for
bibliographic data).

**Licence and attribution.** © N. Srivatsan and T. Berg-Kirkpatrick, "Checklist Models for Improved Output Fluency in Piano Fingering Prediction", in *Proc. of the 23rd Int. Society for Music Information Retrieval Conf.*, Bengaluru, India, 2022 (arXiv:2209.05622; proceedings pp. 525–531, doi:10.5281/zenodo.7316716). Licensed under CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/). This package redistributes the original PDF unchanged (`raw/`), its unedited text extraction (`markdown/`) and an edited chunking (`chunks/`); the changes in the chunking are listed below. It is the only full-text source in this module: the other fingering papers carry no open licence and are cited as bibliography only. This is the FULL-PIPELINE record for this source:
ingest, figure-site census, transcription/formatting, chunking — all in
one pass (both phases landed together).

## Provenance

- Established from the PDF's own front matter: arXiv:2209.05622v1
  [cs.LG], 12 Sep 2022; N. Srivatsan (Carnegie Mellon University) and
  T. Berg-Kirkpatrick (UC San Diego); published in Proc. of the 23rd
  Int. Society for Music Information Retrieval Conf. (ISMIR), Bengaluru,
  India, 2022. Copyright footer: © N. Srivatsan and T. Berg-Kirkpatrick,
  CC BY 4.0. No DOI stated in the stored arXiv PDF; the ISMIR proceedings
  version (pp. 525–531, Zenodo DOI 10.5281/zenodo.7316716, also CC BY
  4.0, checked 2026-11-23) is recorded in `source.yaml` beside the arXiv
  id. The stored copy remains the arXiv v1 file.
- Born-digital with a native text layer (letter/A4 portrait, 7 pages,
  866 KB). ISMIR two-column layout.
- `raw/` holds a byte-identical copy of the original arXiv download
  (SHA-256 `7eba24b8…d372`, recorded in `source.yaml`); never edited.

## Markdown conversion

- Converted 2026-11-11 with a PDF text extractor (markdown output, native
  text-layer PDF path, no forced OCR), its text written verbatim to
  `markdown/srivatsan_2022_checklist_fingering.md` (exit 0).
- Markers are the extractor's native `<page n="N" i="i" />`; no
  adapter applied.
- Marker authority: exactly 7 `<page n="N" i="i" />` markers, one per
  physical page; `n` = printed label 1-7, `i` = 0-based physical index
  0-6, no label reset. 12 `<figure-region>` markers: i=0 rotation ccw
  (the vertical arXiv sidebar stamp) + music ×2 + vector (Figure 1
  residue); i=1 music (onset/offset-time formula residue); i=2 music
  (checklist-function formula residue) + vector (Figure 2); i=3 music
  (REINFORCE loss formula residue) + vector (Figure 3); i=4 render
  (Table 1); i=5 rotation cw (confusion-matrix axis labels) + vector
  (Table 2 region).
- Extraction noise preserved verbatim in `markdown/` (extraction
  evidence, unedited). Known extraction artifacts, repaired only in
  `chunks/` (the edited layer), each noted at its site:
  - The abstract tears mid-sentence (band split): "which in prior work
    using / ings over individual keys" — the print's text layer
    (pdftotext -layout) carries the missing clause "neural models have
    often taken the form of one-hot encod-".
  - Two-column band-splitting interleaves prose around Tables 1-3 and
    Figure 4; the layout text layer restores reading order.
  - Figure 1's subscript labels (x/y with numeric subscripts) extract
    as Odia letters (ସ ଵ ଶ ଷ ଺ ହ) — the print's font maps glyph IDs the
    extractor mis-decodes; no control characters are involved.
- Text-layer control-character check (I01's U+0001/U+0002 class): NONE
  in either the polyplex extraction or pdftotext (checked 2026-11-11;
  the only non-ASCII content is the Odia glyph mis-mapping above plus
  ordinary typographic quotes/dashes/math glyphs).

## Figure-site census (run FIRST)

- `fig.census` over pages 1-7 before any transcription spend. Run id
  `ops-fig-census-20261111T003728Z-4992088b` (exit 0): 7 pages, **8
  sites, 2 recorded skips** (pages 2 and 7, each with prose-content
  evidence — no poison pages, no silent skips).
- Site classes: 4 table (all high except Figure 4's matrix at medium),
  2 pitch-bearing (pages 1 and 4 — both are keyboard/architecture
  diagrams, not staves), 2 decoration low (page-1 copyright footer,
  page-3 figure region the census mis-classed).
- Confidence: 3 high, 3 medium, 2 low.

## Coverage ledger (8 sites + 2 skips — every site resolved)

| Census site                                | Content                                                                                | Representation                                                       | Route                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------ | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| p1 640,385,435,320 (pitch-bearing, medium) | Figure 1 — checklist model architecture diagram                                        | structured description (chunk 01)                                    | ops fig.describe --class diagram (run ops-fig-describe-20261111T004215Z-5ed44dbd, described medium-high) + text-layer labels                                                                                                                                                                                                                                                          |
| p1 115,1452,82,45 (decoration, low)        | copyright/CC-BY licence footer block                                                   | prose restored verbatim in chunk 01                                  | text layer (footer carries a full text layer)                                                                                                                                                                                                                                                                                                                                         |
| p3 120,108,480,112 (decoration, low)       | Figure 2 — input-representation panels (census mis-classed it decoration)              | structured description (chunk 03)                                    | ops fig.describe --class diagram (run ops-fig-describe-20261111T004329Z-545401f4, described medium-high) + text layer                                                                                                                                                                                                                                                                 |
| p4 135,85,455,195 (pitch-bearing, medium)  | Figure 3 — fingering-pattern keyboard diagrams                                         | structured description (chunk 04)                                    | ops fig.transcribe resolution=fine FIRST (run ops-fig-transcribe-20261111T004152Z-b0b432f7, crop ×2-scaled for the 300-dpi render) — honest `unreadable-placeholder`: the site carries no staves (keyboard grids + hand illustration + ∆p/∆f = 6/3 ratio); then ops fig.describe --class diagram (run ops-fig-describe-20261111T004416Z-2d3c33ba, described medium-high) + text layer |
| p5 185,95,910,205 (table, high)            | Table 1 — Bi-LSTM input-representation results                                         | restored markdown table (chunk 06)                                   | text layer (pdftotext -layout); bold reconstructed deterministically from the caption's own per-column rule (Note ⊕ Octave closest on Thumb Cross, Crossed Chord, Step Spread; Lattice closest on the other six) — flagged as a derivation, not a print read                                                                                                                          |
| p6 210,100,830,335 (table, high)           | Table 2 — autoregressive/checklist results                                             | restored markdown table (chunk 06)                                   | text layer; header repaired from the layout columns (the extraction merged "Smear Step Spread Chord Spread" and "Mgen 4-gram" into single cells)                                                                                                                                                                                                                                      |
| p6 355,680,260,195 (table, medium)         | Figure 4 — label-confusion matrix (no printed caption; surrounding prose serves as it) | restored markdown table (chunk 06)                                   | text layer + arithmetic closure: every column sums to ≈100 (100.1 / 100.0 / 100.1 / 100.1 / 99.9 — column-normalized percentages, consistent with one-decimal rounding); rows sum to 117.3 / 114.4 / 86.1 / 67.8 / 114.6 (rows are NOT normalized — recorded)                                                                                                                         |
| p6 115,1140,485,110 (table, high)          | Table 3 — human evaluation ratings                                                     | restored markdown table (chunk 06)                                   | text layer                                                                                                                                                                                                                                                                                                                                                                            |
| p2 (skip)                                  | prose-only: §2 Related Work, §3 Model                                                  | confirmed no content (census evidence retained; text layer complete) | census skip evidence                                                                                                                                                                                                                                                                                                                                                                  |
| p7 (skip)                                  | prose-only: §7 References [1]-[21]                                                     | confirmed no content (census evidence retained; text layer complete) | census skip evidence                                                                                                                                                                                                                                                                                                                                                                  |

**No LilyPond blocks: none of the 8 sites is staff notation.** The
paper carries no music examples with finger numbers — its two
pitch-bearing census sites are the Figure 1 neural-architecture
diagram and the Figure 3 keyboard-pattern grids (the fig.transcribe
run's own anchors name "no staves, noteheads, rhythmic values …"). The
task's LilyPond route is therefore empty by print fact, not by
omission; formulas are restored as inline text from the layout layer
instead.

Tally: 8/8 sites resolved (4 restored tables incl. the confusion
matrix, 3 structured descriptions with the transcribe-first attempt
recorded, 1 prose footer); 2 skips confirmed prose-only. No site
silently omitted.

## Formula restoration (text-layer primary)

The print's math survives in the text layer with two glyph quirks,
both restored in `chunks/` at their sites: the ≠ glyph extracts as
"6=" in the layout text (reward function), and stacked fractions /
∏/Σ constructions linearize (the two factorized likelihoods, the
checklist embedding, the REINFORCE loss). All restored formulas:
p(y|x; θ) = ∏ p(y_i|x; θ) and ∏ p(y_i|y_<i, x; θ); c_i(x_≤i, ŷ_<i) ∈
R^d; f_i(x_<i, y_<i) ⊕ b_i(x_≥i); L = (r̄ − r(x, ỹ)) Σ log p(ỹ_i|ỹ_<i,
x; θ); r(x, y) = exp(−#{i : x_i ≠ x_{i+1}, y_i = y_{i+1}} − #{i : y_i
= y_{i+1}, t⁺*i > t⁺*{i+1}}).

## Known losses

- Figure 1's checklist-keyboard key labels are only partially legible
  (text layer "I II III" + glyph-mis-mapped residue; the describe run
  reads them low-legibility). The certified content is the describe
  run's x₁-x₆ / y₁-y₄ labels plus the text-layer row labels.
- Figure 3's right-hand numerals in the Hop and Crossed Chord panels
  are flagged uncertain by the describe run; the text layer's numeral
  sequence (III I / III III / III II / I III II / I IV / I I / I IV)
  is the primary instrument for printed glyphs, with the uncertain
  Crossed Chord grouping recorded at the chunk site.
- Figure 2's keyboard glyphs carry no printed key names (describe-run
  anchor); the example's note identities beyond the printed box values
  (55, G, 3, +4, −1, +2) are not asserted.
- The reference list's [6] author string and [7] page range ("107 –
  87") are the print's own tokens, restored verbatim (not normalized).

## Chunked layout

- `chunks/` holds the authored chunking (2026-11-11): 7 section-grain
  chunks in reading order, boundaries at the paper's own section
  headings; markers stay inline and untouched, page order is the
  marker authority:
  1. `01_title_abstract_introduction.md` — title page, abstract,
     Figure 1, §1 Introduction (i=0 and i=1 top)
  2. `02_related_work.md` — §2 Related Work (page 2)
  3. `03_model_and_input_representations.md` — §3 Model, 3.1 Input
     Representations, Figure 2, 3.2 Checklist Formulation (page 3)
  4. `04_learning_and_evaluation.md` — §4, 4.1 Fluency Metrics +
     Figure 3, 4.2 Loss Functions + REINFORCE loss and reward (i=3 and
     i=4 top)
  5. `05_experimental_setup.md` — §5, 5.1 Implementation Details, 5.2
     Data (page 5 left column)
  6. `06_results.md` — §6 Results with Table 1, Table 2, Figure 4
     confusion matrix, Table 3 (i=4 right column through i=5)
  7. `07_references.md` — §7 References (i=6)
- `markdown/` remains the unedited extraction evidence; `chunks/` is
  the edited layer (reading order restored from the layout text layer,
  figures resolved per the coverage ledger). Marker multiset across
  `chunks/*.md` equals `markdown/`'s (7 page + 12 figure-region),
  verified at the commit boundary.

## Lint

- `chunks/.markdownlint-cli2.jsonc`: markdownlint-cli2, 7 chunks,
  0 issues, exit 0. Carve-outs (each reasoned in the config): MD013/
  MD060 off (print tables), MD033 allowed `page` + `figure-region`
  (marker contract), MD041 off (marker-first chunk openings), MD040
  off (verbatim figure-region fences carry no language), MD012 off
  (extraction page-join blank lines), MD036 off (the print's caption
  styling).

## Names

The source directory and chunk files were renamed to snake_case when the source moved into this module (`srivatsan-2022-checklist-fingering` → `srivatsan_2022_checklist_fingering`, `01-title-abstract-introduction.md` → `01_title_abstract_introduction.md`, and so on); the content is unchanged.
