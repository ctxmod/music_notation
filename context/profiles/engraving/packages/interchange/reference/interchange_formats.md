# Interchange formats and music fonts

An engraving source rarely lives alone: material arrives from recognition
services and other editors as MusicXML or MEI, and results leave the same
way. This file records where each standard stands and what a transfer
through it can be trusted to carry. The doctrine is constant across all
of them: a converted file is an **import** — "basis: import" until its
text has been checked against a first edition or autograph (P-03) — and
a conversion is verified by a semantic diff, not by its own round trip
(P-09, P-16).

## The standards and their status

All four are maintained outside any single vendor; the W3C ones are
Community Group reports, not W3C Recommendations.

| Standard | Current version | Status (as of 2026-11) | Role |
| --- | --- | --- | --- |
| MusicXML | 4.0 (Final Community Group Report, June 2021) | 4.1 in progress as a draft Community Group specification (milestone reported 66% complete, 2026-10-15) | the de facto interchange format between notation programs and recognition tools |
| MNX | 1.0 draft | under active development in the W3C Music Notation Community Group (specification meetings through 2026-11); the group states it has no stable standard for implementation yet | JSON successor design, semantic-first, separating music from layout so the same content can reflow |
| MEI | 5.1 (2025-01-23) | maintenance release after 5.0 (2023-09); MEI 6 in planning | scholarly encoding: sources, variants, editorial apparatus, facsimile links |
| SMuFL | 1.4 (Final Community Group Report, 2021-02-14) | 1.5 editor's draft | code-point layout and metadata for music fonts |

Bravura, the SMuFL reference font (SIL Open Font License 1.1), is at
1.482 (2026-08-24); the 1.480 release notes say work toward SMuFL 1.5
is under way and that Bravura is meant to reach 1.500 together with it.

## Choosing and trusting a path

- **MusicXML** is the path of least resistance and the one recognition
  tools emit. It carries pitch, rhythm, voices, most notations and
  layout hints well; what survives an import into a particular program
  is that program's interpretation, so check voices, ties across
  barlines, grace types, slur ends and fingering placement against the
  print after every import (the transcription profile's voice
  decomposition decides the voice count, not the importer).
- **MNX** is for prototypes and for following the standard's
  development, not for an edition's archival copy, until a stable
  version is published.
- **MEI** is the format when the apparatus itself is the deliverable —
  readings per source, editorial interventions with responsibility,
  links back to facsimile zones — because it encodes the edition's
  decisions, not only its notes. An MEI export of an edition is
  generated from the edition's records (P-21), never hand-maintained
  beside them.
- **LilyPond** reads MusicXML through `musicxml2ly`; the result is a
  starting draft, re-authored into the house conventions (absolute pitch
  with `\fixed`, bar checks, a dedicated `dynamics` variable) before it
  becomes notation source (P-07).
- **Fonts.** A SMuFL-compliant font lets one glyph name mean the same
  symbol in every program; record the font and its version with the
  edition, because glyph designs change between font releases and a
  rendered-image comparison (P-15) depends on them.

## Sources

- https://www.w3.org/2021/06/musicxml40/ — MusicXML 4.0 Final Community Group Report (June 2021); accessed 2026-11-23.
- https://w3c-cg.github.io/musicxml/ — MusicXML 4.1 draft Community Group specification; https://www.w3.org/community/music-notation/2026/10/15/musicxml-progress-update-october-15-2026/ — 4.1 milestone progress (2026-10-15); accessed 2026-11-23.
- https://w3c-cg.github.io/mnx/docs/ — MNX 1.0 draft specification ("still a work in progress without a stable standard for implementation"); https://www.w3.org/community/music-notation/ — Community Group posts through the MNX specification meeting of 2026-11-03; accessed 2026-11-23.
- https://music-encoding.org/update/2025/01/23/MEI-51-released.html — MEI 5.1 release (2025-01-23), maintenance focus, MEI 6 outlook; accessed 2026-11-23.
- https://w3c-cg.github.io/smufl/latest/ — SMuFL 1.5 (draft); the Community Group's final-reports list gives SMuFL 1.4 (2021-02-14); accessed 2026-11-23.
- https://github.com/steinbergmedia/bravura/releases — Bravura 1.482 (2026-08-24), 1.480 notes on the path to SMuFL 1.5; accessed 2026-11-23.
