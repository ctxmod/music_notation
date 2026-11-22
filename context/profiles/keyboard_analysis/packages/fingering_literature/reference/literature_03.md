# Literature Index: Automatic Piano Fingering, Hand Division, Span, Pedalling — part 3 of 6

Continued from `literature_02.md`.
## 3. Randolph 2023, "Toward Actionable Computational Models of Piano Fingering"

**Citation.** David A. Randolph, PhD dissertation, University of Illinois Chicago, 2023 (advisor B. Di Eugenio). It incorporates SMC 2021 (ERR), SMC 2023 (CRF), ISMIR LBD 2016 (abcDE, Dactylize) and ISMIR LBD 2018 (Pydactyl). Chunks: Randolph 2023.

### 3.1 Problem taxonomy
part 02, "ch1 introduction" §1.3, Table I
Each axis runs from a basic setting to a mature one; the category column says whether it constrains input or output.

| # | Basic | Mature | Category |
|---|---|---|---|
| 1 | unimanual | bimanual | output |
| 2 | monophonic | polyphonic | input |
| 3 | restricted | unrestricted (e.g. allows repeated notes) | input |
| 4 | **segregated** (staff = hand) | **integrated** (a hand plays notes from the other staff) | input |
| 5 | normalized | dynamic (velocity) | input |
| 6 | quantized | articulated (real durations) | input |
| 7 | atomic | substitutional | output |
| 8 | non-pedaled | **pedaled** | input/output |

- Randolph states that **no published model uses or outputs pedalling, and none outputs substitutions**.
- The damper and sostenuto pedals are an alternative to finger legato and relieve constraints when notes must sound long. A mature model should give pedalling advice coordinated with fingering.
- **Table II** (part 03, "ch2 related work") classifies every system on these axes. All are segregated except De Prisco 2012 and Randolph's CRF. Only the CRF is articulated and dynamic. None is pedalled or substitutional.
- **Table III** compares piano and guitar fingering systems by ergonomic, cognitive and interpretive factors, melody, chords and validation.

### 3.2 Related-work digest
part 03, "ch2 related work" §2.2–2.4

| Year | Authors | Approach |
|---|---|---|
| 1997 | Parncutt et al. | Ergonomic rule model (§1). |
| 1998 | Viana et al. | An expert system of common rules, falling back to a genetic algorithm. Fitness is total "fingering distance". |
| 2000 | Hart, Bosch, Tsai | DP over "basic movements". Four lookup tables, one per key-colour transition. Each has 14 finger combinations × intervals of 1–12 semitones = 168 entries, 672 in all. Difficulty ratings are user-revisable (§3.6). |
| 2001 | Jacobs | Refinements to Parncutt: physical distances instead of semitones, modified heuristics. Under ERR it is not significantly better than Parncutt. |
| 2005/2007 | Al Kasimi et al. | Trellis with vertical and horizontal costs (§5). |
| 2006 | Lin & Liu | A bit-wise fast Parncutt for a real-time tutor. Not reproducible. |
| 2007 | Yonebayashi et al. | HMM, melodies only. |
| 2014 | Nakamura et al. | Merged-output HMM for both hands. Chords are treated as fast arpeggios. RH 56.0% / LH 55.4% in cross-validation. |
| 2015/2017 | Balliauw et al. | Tabu search, then variable neighbourhood search, polyphonic. Objective based on Parncutt plus Jacobs plus polyphony additions. Evaluated only anecdotally. |
| 2012 | De Prisco et al. | Differential evolution with an ANFIS fitness learned from triples of fingering changes. Reports 87.3% "accuracy" on piano by an expert's judgement of 2 pieces. |
| 2021/2022 | Guan et al. | BLSTM on pitch differences, with synthetic PIG augmentation from an HMM, and **post-hoc ergonomic rules**. Claims +4.06% average match and +2.87% highest match over the 3rd-order HMM. Introduces the "irrational fingering rate" (IFR). |

### 3.3 Annotation format: abcD / abcDF
part 04, "ch3 data collection systems" §3.1
- **abcD** is valid abc with a header block, for example:
  `% abcD fingering 1: 1/23231313123-14534@3131`
  - The RH staff comes first, then `@`, then the LH.
  - `1/2` means substitution or alternative.
  - Authority, transcriber and date lines follow.
- **abcDF EBNF grammar** (Listing 3.2):
```
sequence := staff ("@" staff)?
staff := line "&" staff | line | ""
line := (score_fingering)*
score_fingering := ornamental ("/" ornamental)? | pedaled_fingering ("/" pedaled_fingering)? | pedaling ("/" pedaling)?
ornamental := "(" (pedaled_fingering)+ ")"
pedaling := (soft)+ "x" (damper)+
pedaled_fingering := (soft)? fingering (damper)? (segmenter)?
fingering := finger ("-" finger)?        # "-" = substitution (strike-release)
finger := (hand)? digit                  # "<" / ">" switch hand (integrated fingering)
segmenter := "," | ";" | "."             # 3 levels of phrase boundary
damper := "_" | "^"                      # damper pedal events
soft := "p" | "f"                        # soft pedal events
digit := "1".."5"
```
- This is **the only format in these sources that encodes damper and soft pedalling, substitutions, cross-hand notes and phrase segmentation alongside fingering**. It is a ready-made design reference for the crate's annotation I/O.
- The exact meaning of `_` versus `^` is not spelled out in the extracted text; check the abcDE docs before relying on it.
- **Dactylize** (§3.2) is about $300 of hardware: foil on the keys and conductive tape on the fingers form a patch-bay matrix polled by an Arduino and synced to MIDI. It captures finger, note, velocity and pedal.

### 3.4 Corpora
part 05, "ch4 corpus development"; all at `github.com/dvdrndlph/didactyl`
- **Beringer corpus:** Beringer & Dunhill scales, arpeggios and broken chords in 12 keys for both hands. **11,860 notes over 194 exercises**, fully fingered including alternates. Missing digits were filled in by 2 pianists, who agreed perfectly. This is a **canonical standard-fingering table** a deterministic engine could match or embed.
- **Parncutt corpus:**
  - **P_pub:** the original 28 pianists, 45 notes, 1260 tags.
  - **P_new:** **190–206 pianists** fingered *extended* versions of the 7 Czerny fragments in both staves: 185 notes, 36,185 tags (Table IV).
  - Fingerings are linked to player profiles.
- **WTC corpus:** sparse fingerings for Bach WTC selections from 29 pianists, plus the Czerny and Tovey/Samuel editions.
- **Layer One (L1):** first sections of Clementi op. 36 sonatinas. Sparse fingerings from 6 editions plus full fingerings from 6 professionals ("L1 full"). Phrase-segmented.
- **Phrase agreement:** upper staff κ ≈ 0.879 (1255/1262 notes); lower staff κ ≈ 0.410. Phrasing as an input is useful mostly for melody.
- **Representation notes:**
  - Staff assignment has historically suggested which hand plays. Real practice deviates as repertoire gets harder (fn on p. 75).
  - PIG separates by hand rather than staff, so it solves a "notation-free" sub-problem.

### 3.5 ERR evaluation
part 06, "ch5 err evaluation"
- **Framing:** fingering as **information retrieval**. A system returns a ranked list of fingerings; the user stops at the first acceptable one.
- **Metric:** ERR(R) = Σ_{r=1..R} (1/r) · ∏_{i<r} (1 − P_i) · P_r, with **P_r = 1 − Δ(H, S_r) / N**, where H is the human's fingering and S_r is the suggestion at rank r.
- **Choices for Δ:**
  - Hamming distance.
  - **"AdjLong"** weighted unigram distance: 2↔3 and 3↔4 cost ½; every other mismatch costs 1 (Table V).
  - **Trigram τ:** a note counts as matched only if its whole 3-note window matches.
  - **"Nuanced" τ:** a middle-note 2↔3 or 3↔4 swap costs 1 − ε when the outer notes match.
  - **"Relaxed" τ:** the nuanced discount applies at any position in the trigram. The experiments use ε = 0.99.
- **mERR** is the mean over annotators.
- **Validation** on P_pub and P_new, with lists of length 5 (Table VIII). The models rank Ideal > {Jacobs ≈ Parncutt} > Random under every Δ; Jacobs and Parncutt are not significantly different. Selected rows:

| Model | Hamming | AdjLong | Relaxed |
|---|---|---|---|
| IdealNew | 0.811 | 0.857 | 0.738 |
| Jacobs | 0.670 | — | — |
| Parncutt | 0.664 | 0.759 | 0.540 |
| RandomNew | 0.403 | — | 0.133 |

  Trigram measures separate good from bad models about 50% more widely.
- **Polyphony:** one-hand chord fingering is "unambiguously constrained" to ascending finger order (RH). Chord fingering is easier than melodic fingering.

### 3.6 Pydactyl toolkit and reference models
part 07, "ch6 piano fingering toolkit"
- **Framework:** Python, built on music21. A `Dactyler` base class handles segmentation.
- **Combining segment advice is a k-shortest-path problem.** Segment cost can be the raw sum, normalised by length, or by ordinal rank.
- **Evaluation distances:** Hamming; **"natural"** = |finger difference|, plus 10 extra across hands (Table IX); and **"reposition-penalizing"**, where a thumb mismatch with another finger in the same hand costs 4–7 (Table X, below).

| | 1R | 2R | 3R | 4R | 5R |
|---|---|---|---|---|---|
| **1R** | 0 | 4 | 5 | 6 | 7 |
| **2R** | 4 | 0 | 1 | 2 | 3 |
| **3R** | 5 | 1 | 0 | 1 | 2 |
| **4R** | 6 | 2 | 1 | 0 | 1 |
| **5R** | 7 | 3 | 2 | 1 | 0 |

  Cross-hand costs run 10–18.
- **Other evaluations:** a pivot-alignment metric, and **"re-entry cost"** (re-advise from the first deviation, with the first finger constrained). Every model must support constraining the first and last finger of a segment.
- **Enhancements to all reference models:**
  - **repeated notes are allowed and must reuse the same digit**
  - RH rules mirrored for the LH
  - chords arpeggiated ("quasi-polyphonic")
  - still segregated
- **Parncutt reference:** trigram nodes; 1-note rules applied only to the middle note and 2-note rules only to the first pair, to avoid double counting; uniform weights; Dijkstra. Jacobs' variant is also included.
- **Hart model:** 4 key-colour tables × 14 finger combinations × 12 intervals; DP. The text says the tables are white-to-white, white-to-black, black-to-black and "white-to-white", the last presumably black-to-white.
- **Sayegh 1989 "optimum path paradigm":** learn arc weights as frequencies, adding 1/|E| along each training example's optimal path, then take the *maximal* path (Listing 6.2).

### 3.7 CRF model
part 08, "ch7 crf model"
- **Model:** first-order linear-chain CRF (sklearn-crfsuite, L-BFGS, defaults c1 = 0, c2 = 1, no tuning).
- **Total order:** onset, then ascending pitch.
- **Features** (all functions of the observed notes):
  - **Distance:** lattice dx and dy to each of the ±3 neighbours (12 numbers); dx is **capped at 15**. Concatenated dx n-grams over 3, 5 and 7 notes; a leap flag.
  - **Concurrency:** counts of lower and higher notes with onsets within **30 ms**, concatenated with the staff; a chord-border flag; counts of lower and higher notes sounding over the attack by more than 30 ms. This assumes no damper pedal and is a generalised "vertical cost".
  - **Segment boundary:** start and end flags.
  - **Black key:** current, previous and next notes black, plus a trigram of colours.
  - **Dynamics:** MIDI velocity.
  - **Articulation**, over a 9-note window:
    - number of notes followed by silence ≥ ½ their own duration
    - normalised silence (total inter-note silence divided by the window's onset span)
    - number of notes separated by ≥ **60 ms**
  - **Tempo:** attacks per second over the previous 5 notes, the next 5, and the 9-note window.
  - **Repetition:** how many times the same pitch is struck immediately before and after.
- **Five-fold grouped cross-validation** (Table XIII):

| Data | Accuracy | M_gen |
|---|---|---|
| Arpeggio | 0.858 | 0.877 |
| Broken chord | 0.855 | 0.874 |
| Scale | 0.692 | 0.707 |
| L1 full | 0.509 | 0.507 |
| L1 full + Beringer | — | 0.798 (+29 points; **training on rudiments generalises**) |
| PIG training | 0.669 | 0.664 |
| All data | 0.719 | 0.764 |

- **Head-to-head on the PIG test set** against the 3rd-order HMM (Table XIV, unweighted):

| | CRF | HMM |
|---|---|---|
| M_gen | 0.6455 | 0.6448 |
| M_high | 0.6849 | 0.7100 |
| M_soft | 0.8515 | 0.8549 |

  None of the differences is significant (p > 0.069). Per-segment scores are in Tables XVI and XVII.
- **Critique of channel separation:** match rates are valid only when the hand assignment is uncontroversial. PIG gives one arbitrary hand-separated representation per piece, while scores imply staff = hand.

### 3.8 Hand-reach survey
part 11, "appendix b", Figs 81–86. **N = 198** pianists.
- **Self-rated hand size** vs non-pianists: very small 5%, small 21%, average 42%, large 27%, very large 6%.
- **RH, thumb on C4, highest white key reachable with finger 5:**

| Note (interval) | C5 (octave, 12 st) | D5 (9th, 14 st) | E5 (10th, 16 st) | F5 (11th, 17 st) | G5 (12th, 19 st) | above G5 |
|---|---|---|---|---|---|---|
| **Practically** | 19% | **49%** | 28% | 3% | 0.5% | — |
| **Possibly** | 3% | 31% | **49%** | 16% | 1% | 0.5% |

- **LH practical**, thumb on C4 down to: D3 (7th) 0.5%, C3 (octave) 18%, B2 (9th) 46%, A2 (10th) 31%, G2 (11th) 4%.
- **LH "possible" is narrower than LH "practical"**: C3 33%, B2 45%, A2 17%. This is inconsistent with the RH pattern and probably a labelling or response artefact in the source.
- **[derived]** These distributions bracket Parncutt's MaxPrac(1,5) = 15 semitones, with a median practical reach of a 9th to a 10th. They are good priors for a hand-size parameter: small ≈ octave, medium ≈ 9th, large ≈ 10th.
- **Other survey items** (Figs 41–44): starting fingers for C, A♭, E♭ and B♭ major scales, use of conventional scale and arpeggio fingerings, wrist and finger posture, pain history. Fingering data is linked to these profiles.

### 3.9 Conclusions and limitations
part 09, "ch8 conclusions"
- Multiple ground truths in training data place an upper bound on any model.
- The recommendation is per-pianist ("archetype") models, collected with Dactylize, combined in a **recommender** that ranks diverse suggestions and is evaluated with mERR.
- Guan's post-hoc ergonomic rules and synthetic data look promising. Randolph suggests **post-processing with rule-based models, with first and last fingers constrained**.
- Repeated warning: the PIG data are humanized/synthetic and not tied to scores.

### 3.10 Implementation notes for a Rust deterministic engine
- **abcDF is the template for an annotation and output format**: fingering, substitutions, hand-switch markers, phrase segmenters, damper and soft pedal in one string per staff. Recommend supporting it, or a superset, for round-trip tests.
- **Reusable as is:**
  - the Parncutt span tables and rules with Randolph's errata and two-sided pruning
  - repeated note ⇒ same finger as a default
  - the Hart key-colour × interval × finger-pair table as a natural data structure for a tunable deterministic cost (the actual numbers are not in the repo; acquire Hart 2000)
- **Evaluation:** implement M_gen, M_high, M_soft, M_rec, the AdjLong and trigram distances and ERR, so the crate can score k-best output against corpora such as Beringer, the Parncutt corpus and PIG.
- **The CRF feature set doubles as a list of deterministic heuristic inputs:** 30 ms chord grouping, 60 ms separation, staccato ratio, attack rate as tempo proxy, repetition counts, black-key trigram, velocity.
- Beringer scales and arpeggios are a must-pass regression suite for any engine.

---
