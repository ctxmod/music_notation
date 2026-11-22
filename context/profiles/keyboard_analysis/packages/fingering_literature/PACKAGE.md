---
description: >-
  The research literature on automatic piano fingering, hand separation,
  hand span and pedalling, synthesised across sources, with the CC BY 4.0
  Srivatsan and Berg-Kirkpatrick (2022) paper stored as a source.
binds_to_depth: main...
---

# Fingering literature

A literature index for designing a deterministic analyser of piano scores (hand separation, stretch and span analysis, pedalling, fingering generation, MIDI), written so an architect can work from it without opening the papers. Statements marked *[derived]* are the indexer's own inference, not something a source states.

Reading order:

1. `reference/literature_05.md` §6: the cross-source synthesis (consensus constraint model, polyphony, hand separation, pedal, legato, evaluation stack).
2. The per-paper files as needed:

| File | § | Covers |
| --- | --- | --- |
| `reference/literature_01.md` | 0–1 | Quick map; Parncutt et al. 1997: span terminology and the six-level finger-pair span table, the 12 rules with formulas, the dynamic program over 3-note states, Czerny data, errata, implementation notes |
| `reference/literature_02.md` | 2 | Nakamura, Saito, Yoshii 2020: the PIG dataset and format, HMMs of orders 1–3, the chord HMM, the 30 ms chord rule, match-rate metrics, results |
| `reference/literature_03.md` | 3 | Randolph 2023: model taxonomy, the abcDF annotation grammar (encodes pedal, substitutions and cross-hand notes), corpora, the ERR ranked-list metric, Pydactyl reference models, the CRF, the hand-reach survey of 198 pianists |
| `reference/literature_04.md` | 4–5 | Srivatsan & Berg-Kirkpatrick 2022: checklist models and deterministic fluency checks. Al Kasimi, Nichols, Raphael (AFS): chord trellis, in-chord and between-chord costs |
| `reference/literature_05.md` | 6–8 | Cross-source synthesis; study-library passages on fingering, pedalling, legato and hand use (Hofmann, Arrau, Schmitt, Kullak, Taub, Walker and others); bibliography to acquire next |
| `reference/literature_06.md` | 9 | Design notes for the architect *[derived]* |

Only `sources/srivatsan_2022_checklist/` carries full text (CC BY 4.0, © N. Srivatsan and T. Berg-Kirkpatrick; attribution in its `SOURCE.md`). The other papers and the pedagogical books are cited as bibliography only.
