# Optical music recognition: what it gives a transcription

Recognition output is where most transcriptions start and where their
most expensive errors come from. This file records what current optical
music recognition (OMR) can and cannot be trusted to deliver, so the
reading discipline (`voice_decomposition.md`,
`rules/transcription_context_discipline.md`) is aimed at the right
places. Whatever produced it, recognition output is an **import**
(edition principles P-03, P-05): it becomes text only after it has been
read against the print, measure by measure.

## Where the field stands (2026)

- **End-to-end models have overtaken staged pipelines on typeset
  music.** The research front moved from staff-removal → symbol
  detection → reconstruction pipelines to image-to-sequence transformers
  that emit a whole page as a symbolic encoding. The Sheet Music
  Transformer line (Ríos-Vila et al., full-page pianoform OMR,
  *International Journal of Computer Vision*, 2025) transcribes full
  piano pages and reports outperforming a commercial tool zero-shot and
  after fine-tuning; LEGATO (Yang et al., ICLR 2026) is a large
  pretrained model emitting ABC notation for full- and multi-page typeset
  scores, reporting large error reductions on the TEDn and OMR-NED
  metrics.
- **Claims are about typeset scores.** The headline results are on
  engraved (often synthetic or cleanly typeset) music; handwritten
  manuscripts, early prints, dense orchestral pages and degraded scans
  remain markedly harder. A model's benchmark number says little about a
  particular nineteenth-century plate.
- **Open-source engines exist and emit MusicXML.** Audiveris (current
  release 5.11.0, 2026-07-11) is the long-standing open engine with an
  editing UI; its output, like every engine's, goes through the import
  path in the engraving profile's `interchange` package.
- **Evaluation metrics are tree- and sequence-edit distances** (TEDn over
  the MusicXML tree, OMR-NED over normalised sequences). They count
  symbol-level edits, not musical consequences: one wrong voice
  assignment can cost few edits and many downstream decisions.

## Where recognition fails, so where reading starts

Spend the gridded-crop budget where recognisers are systematically weak. The table is a working synthesis of recurring recognition errors, not a measured benchmark; extend it from the edition's own import diffs:

| Feature | Typical failure |
| --- | --- |
| Voices and shared noteheads | voices merged into chords, or a shared head read twice; decide the count first (`voice_decomposition.md`) |
| Ties versus slurs | ties read as slurs and vice versa, especially across barlines and system breaks |
| Accidentals | courtesy and carried-through accidentals dropped or doubled; key changes mid-system |
| Grace notes | kind lost (slash, slur), or graces absorbed into the main rhythm |
| Tuplets and irregular groups | missing brackets turn into wrong durations that still fill the bar |
| Cross-staff beams, kneed beams, hand crossings | notes moved to the wrong staff |
| Fingering, pedal, text spanners | dropped entirely or attached to the wrong note |
| Dynamics and hairpins | placement drift between staves |

A bar that "adds up" is not a correct bar: rhythmic completeness is the
check recognisers already optimise for, so a duration error usually
hides inside a full measure.

## Sources

- https://arxiv.org/abs/2405.12105 — A. Ríos-Vila, J. Calvo-Zaragoza, D. Rizo, T. Paquet, "End-to-End Full-Page Optical Music Recognition for Pianoform Sheet Music" (Sheet Music Transformer; latest arXiv version 2025-06-27); published in *International Journal of Computer Vision*, https://link.springer.com/article/10.1007/s11263-025-02654-6 (2025); accessed 2026-11-23.
- https://arxiv.org/abs/2506.19065 — G. Yang, V. Ebert, N. Tamer, L. Pozzobon, N. A. Smith, "LEGATO: Large-scale End-to-end Generalizable Approach to Typeset OMR" (2025; ICLR 2026, https://iclr.cc/virtual/2026/poster/10009474); accessed 2026-11-23.
- https://github.com/Audiveris/audiveris/releases — Audiveris 5.11.0 (2026-07-11); accessed 2026-11-23.
