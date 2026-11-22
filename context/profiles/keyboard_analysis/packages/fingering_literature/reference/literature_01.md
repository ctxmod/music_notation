# Literature Index: Automatic Piano Fingering, Hand Division, Span, Pedalling

Knowledge base for designing a deterministic Rust crate that analyses piano scores (hand separation, stretch/span analysis, pedalling, fingering generation, MIDI). It is written so that an architect can work from it without opening the papers, and every section points at the chunk files holding the detail.

Sources are cited by author and year, with the part of the source a point comes from ("part 03, \"model overview and terminology\""). Only Srivatsan and Berg-Kirkpatrick 2022, licensed CC BY 4.0, is stored with this package, as the source `sources/srivatsan_2022_checklist/`; the other papers and the pedagogical study library are cited as bibliography only, because their text carries no open licence. Numeric tables below were checked against the chunk text. "[derived]" marks values I computed or inferred; everything else is reported by the source.

---

## 0. Quick map

| # | Source | Chunk dir | What it gives the engine |
|---|---|---|---|
| 1 | Parncutt, Sloboda, Clarke, Raekallio, Desain 1997 | Parncutt et al. 1997 | A fully specified **rule-based cost model**: a 6-level span table for each finger pair, 12 weighted rules, and DP over a trigram-state graph. Directly implementable. |
| 2 | Nakamura, Saito, Yoshii 2020 | Nakamura, Saito and Yoshii 2020 | **HMMs of orders 1–3** and a chord HMM, the **PIG dataset** and its format, the **match-rate metrics** (M_gen/M_high/M_soft/M_rec), and state-of-the-art numbers. |
| 3 | Randolph 2023 (PhD dissertation, UIC) | Randolph 2023 | Problem taxonomy (8 "maturity" axes), the **abcDF fingering+pedal annotation grammar**, new corpora, the **ERR evaluation**, Pydactyl reference implementations (Parncutt/Jacobs/Hart/Sayegh), **errata to Parncutt**, a CRF feature set, and **hand-reach survey data**. |
| 4 | Srivatsan & Berg-Kirkpatrick 2022 | `sources/srivatsan_2022_checklist/chunks/` | **Fluency metrics** (thumb/thumbless crosses, crossed chords, hops, smears, step/chord spread) with the human "gold" baselines. Neural checklist and REINFORCE models. Input-representation ablation showing the lattice representation is best. |
| 5 | Al Kasimi, Nichols, Raphael (AFS, ISMIR 2005, venue inferred) | Al Kasimi, Nichols and Raphael 2005 | A **polyphonic chord-state trellis** with **vertical (in-chord) and horizontal (transition) costs**, a per-finger-pair piecewise cost curve set from user calibration, and a sketch of how to learn its parameters. |
| — | Study library (Schmitt, Hofmann, Arrau, Taub, Kullak, Walker/Chopin, ...) | study library | Pedagogical rules for **pedalling** (timing, harmony changes, register), thumb/black-key practice, legato, repeated notes, octaves, chromatic scale, and **hand redistribution**. See §7. |

---

## 1. Parncutt et al. 1997, "An Ergonomic Model of Keyboard Fingering for Melodic Fragments"

**Citation.** R. Parncutt, J. A. Sloboda, E. F. Clarke, M. Raekallio, P. Desain. *Music Perception* 14(4):341–382, Summer 1997. Chunks: Parncutt et al. 1997, part 01, "title abstract" … part 11, "appendix".

### 1.1 Problem framing
part 03, "model overview and terminology"
- **Right hand only. Monophonic, consecutive melodic notes. Short fragments** (the implementation enumerates up to 8 notes; DP extends this to arbitrary length). Fragment length is tied to the sight-reading eye-hand span of about 6 notes, or about 2 s.
- **Input:** a pitch series given as semitone offsets. Duration, articulation and dynamics are ignored; the model assumes **legato isochrony at mezzoforte**.
- **Output:** every fingering that satisfies the constraints, ranked by summed difficulty.
- **Hand separation:** not addressed. RH only. The LH is obtained by mirroring (see 1.3).
- Disallowed: the same finger on consecutive notes, including **repeated notes**; **mute finger changes (substitution)**; and crossing non-thumb fingers over each other. Footnote 6 notes that this last practice was historically used by Chopin and Busoni.

### 1.2 Span terminology
part 03, "model overview and terminology"
- **MaxPoss** is the anatomical limit. Wagner 1988 measured it in more than 200 pianists; the model does not use it.
- **MaxPrac / MinPrac** are the largest and smallest spans used in practice between consecutive legato notes. These are **hard constraints**.
- For thumb pairs, **MinPrac is negative**: it is the farthest a finger can pass over the thumb, or the thumb under a finger.
- **MinRel / MaxRel** define the "relaxed" range. Consecutive fingers fall on consecutive diatonic white keys ("next-note-next-finger").
  - For non-thumb pairs, MaxRel = 2 × (finger difference) and MinRel = MaxRel − 1.
  - For thumb pairs, MaxRel = MaxComf − 3 and MinRel = 2 × (finger difference) − 1.
- **MaxComf = MaxPrac − 2** for all pairs. **MinComf = MinPrac − 2** for thumb pairs and **= MinPrac** for the others.
- MinPrac for thumb pairs is deliberately set **just too small for legato arpeggios**. In C major, 1-under-3 needs 5 semitones while MinPrac(1,3) = −4, so fast arpeggios are not finger-legato.
- **Hand size:** multiply all values by about 0.9 for smaller hands. An octave is about 165 mm, or about **13.7 mm per semitone**, averaged over transpositions; the paper suggests a future model in mm per key.
- **Cortot's virtuoso extension targets** (fn. 3), in semitones and useful as a MaxPoss-like upper bound:
  - 1-2 = 11, 1-3 = 13, 1-4 = 15, 1-5 = 17
  - 2-3 = 8, 2-4 = 11, 2-5 = 12
  - 3-4 = 7, 3-5 = 10
  - 4-5 = 8

**Table 1: RH finger-pair spans in semitones.** For pair f-g, f plays the first note. Positive values are rising intervals; negative values mean passing over or under the thumb.

| Pair | MinPrac | MinComf | MinRel | MaxRel | MaxComf | MaxPrac |
|---|---|---|---|---|---|---|
| 1-2 | −5 | −3 | 1 | 5 | 8 | 10 |
| 1-3 | −4 | −2 | 3 | 7 | 10 | 12 |
| 1-4 | −3 | −1 | 5 | 9 | 12 | 14 |
| 1-5 | −1 | 1 | 7 | 10 | 13 | 15 |
| 2-3 | 1 | 1 | 1 | 2 | 3 | 5 |
| 2-4 | 1 | 1 | 3 | 4 | 5 | 7 |
| 2-5 | 2 | 2 | 5 | 6 | 8 | 10 |
| 3-4 | 1 | 1 | 1 | 2 | 2 | 4 |
| 3-5 | 1 | 1 | 3 | 4 | 5 | 7 |
| 4-5 | 1 | 1 | 1 | 2 | 3 | 5 |

- **Reverse pairs (g-f):** negate the values and swap Min and Max. For example, MaxPrac(2,1) = 5.
- **Left hand:** reverse the finger numbers, or negate and swap. The paper notes that Wagner's data show the hands differ in size.
- **Full 20-row table with reverse pairs:** reproduced as Table XI in Randolph 2023, part 07, "ch6 piano fingering toolkit". For example:
  - 2-1: −10, −8, −5, −1, 3, 5
  - 5-1: −15, −13, −10, −7, −1, 1
  - 4-3: −4, −2, −2, −1, −1, −1

**Appendix: matrix form.** part 11, "appendix". Rows are the first finger f and columns the second finger g, 1–5. Each **Min matrix** is the negated transpose of the matching Max matrix, e.g. MinPrac(f,g) = −MaxPrac(g,f).

MaxPrac:
```
      g=1  2   3   4   5
f=1    0  10  12  14  15
f=2    5   0   5   7  10
f=3    4  -1   0   4   7
f=4    3  -1  -1   0   5
f=5    1  -2  -1  -1   0
```
MaxComf:
```
f=1    0   8  10  12  13
f=2    3   0   3   5   8
f=3    2  -1   0   2   5
f=4    1  -1  -1   0   3
f=5   -1  -2  -1  -1   0
```
MaxRel:
```
f=1    0   5   7   9  10
f=2   -1   0   2   4   6
f=3   -3  -1   0   2   4
f=4   -5  -3  -1   0   2
f=5   -7  -5  -3  -1   0
```

### 1.3 Algorithm
part 03, "model overview and terminology", part 06, "ranking and optimal sequences"
1. **Enumerate** all finger sequences with no immediate repeats: 5 × 4^(n−1) of them. Keep a sequence only if every consecutive interval satisfies MinPrac ≤ i ≤ MaxPrac. For example, C4-E4-G4 has 24 legal fingerings.
2. **Score** each sequence with 12 additive rules (1.4). The score is the sum of rule points, each multiplied by its weight. The paper says the weights were fixed ad hoc. Parncutt later told Randolph that **all weights were equal (1)** (Randolph ch6, fn).
3. **Rank** by total cost.
4. **Longer passages: shortest path through a network.**
   - Nodes are (note, finger) pairs, pruned of dead ends and unreachable nodes.
   - The graph is then **"exploded"** so that each node carries (previous finger, current finger, next finger). This lets the 3-note rules be evaluated locally:
     - 1-note rule (Weak-Finger): current note only.
     - 2-note rules (Stretch, Small-Span, Large-Span, Three-to-Four, Four-on-Black): previous and current.
     - 3-note rules (Position-Change-Count, Position-Change-Size, Three-Four-Five, Thumb-on-Black, Five-on-Black, Thumb-Passing): previous, current and next.
   - The sweep is a Dijkstra-style left-to-right pass. It needs at most n × 5 × 4 × 4 + 2 nodes and runs in roughly linear time.
   - Worked example: A4-C4-A4-E5 has an optimum of **4-1-2-5 at cost 15**.
5. **Boundary constraint:** the first and last fingers may be fixed as an input constraint. The experiment fixed the final finger.

### 1.4 The 12 rules
Rules 1–5 are in part 04, "rules span rules", rules 6–12 in part 05, "rules strength and black keys", and the exact formulas of rules 1–3 in part 11, "appendix". In the table, **i** is the semitone interval from finger f to finger g; ranges come from the f,g matrices.

| # | Rule (abbrev.) | Scope | Points |
|---|---|---|---|
| 1 | **Stretch** (str) | 2 notes | If i > MaxComf: 2·(i − MaxComf). If i < MinComf: 2·(MinComf − i). |
| 2 | **Small-Span** (sma) | 2 notes | If i < MinRel: (MinRel − i) when the thumb is involved, 2·(MinRel − i) otherwise. |
| 3 | **Large-Span** (lar) | 2 notes | If i > MaxRel: (i − MaxRel) when the thumb is involved, 2·(i − MaxRel) otherwise. When f > g the labels of rules 2 and 3 swap; totals are unaffected. |
| 4 | **Position-Change-Count** (pcc) | 3 notes | A change occurs when the interval from note 1 to note 3 lies outside [MinComf, MaxComf] for (finger 1, finger 3). It is a **full change (2 pts)** when the middle finger is the thumb, the middle pitch lies between notes 1 and 3, and the 1→3 interval lies outside [MinPrac, MaxPrac]; otherwise it is a **half change (1 pt)**. Special case: a repeated pitch with a finger change around one intervening note (e.g. C-F-C played 2-5-1) is a half change of size 0. |
| 5 | **Position-Change-Size** (pcs) | 3 notes | How far the note-1→note-3 interval lies outside [MinComf, MaxComf], in semitones. This is economy of movement: the distance the hand's centre of gravity travels. |
| 6 | **Weak-Finger** (wea) | 1 note | 1 point per use of finger 4 or 5. |
| 7 | **Three-Four-Five** (345) | 3 notes | 1 point whenever fingers 3, 4 and 5 occur consecutively in any order (overlapping groups count). |
| 8 | **Three-to-Four** (3t4) | 2 notes | 1 point when 3 is immediately followed by 4. |
| 9 | **Four-on-Black** (bl4) | 2 notes | 1 point when 3 and 4 are consecutive in either order, with 3 on white and 4 on black. |
| 10 | **Thumb-on-Black** (bl1) | 3 notes | 1 point if the thumb plays black; +2 if the preceding note is white; +2 if the following note is white. |
| 11 | **Five-on-Black** (bl5) | 3 notes | 0 points if finger 5 is on black with black neighbours; +2 if the preceding note is white; +2 if the following note is white. |
| 12 | **Thumb-Passing** (pa1) | 2 notes | 1 point per thumb pass on the same level (white→white or black→black); 3 points when the lower note is white on a non-thumb finger and the upper note is black on the thumb. |

**Table 2:** the combined cost of rules 1–3 for every pair and interval (part 04, "rules span rules"). Row 2-4 by interval, from MinPrac to MaxPrac:

| Interval (semitones) | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|
| Cost | 4 | 2 | 0 | 0 | 2 | 6 | 10 |

Rows 1-2 and 1-5 by interval:

| Pair | −5 | −4 | −3 | −2 | −1 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1-2 | 10 | 7 | 4 | 3 | 2 | — | 0 | 0 | 0 | 0 | 0 | 1 | 2 | 3 | 6 | 9 | | | | | |
| 1-5 | | | | | 12 | — | 6 | 5 | 4 | 3 | 2 | 1 | 0 | 0 | 0 | 0 | 1 | 2 | 3 | 6 | 9 |

Asymmetry: for thumb pairs, MinPrac is costlier than MaxPrac. For the other pairs the reverse holds.

**Supporting ergonomic facts** (part 05, "rules strength and black keys"):
- **Kullak's 5 hand positions:** all fingers on white is easiest. All on black is slightly harder. 1 and 5 on black with the others on white is **hardest**.
- **Thumb-pass difficulty**, easiest first:
  1. non-thumb on black ↔ thumb on white
  2. white → white
  3. black → black
  4. non-thumb on white → thumb on black, which is by far the hardest; in practice legato there comes from the pedal, e.g. Chopin op. 25/11
- **Sight-reading corpus** (Sloboda et al., ≈30k fingered notes, 17 pianists, Czerny):
  - The transition 4→3 occurred **1663** times and 3→4 **1142** times, i.e. 4→3 was 45% more frequent.
  - Other significant asymmetries, not modelled: 3-1 > 1-3, 1-4 > 4-1, 3-2 > 2-3, 2-4 > 4-2, 5-4 > 4-5.
- **Why 1 and 5 on black keys is avoided:** short fingers on short keys pull the hand forward. The leverage at the key fulcrum also affects evenness, and the effect is strongest in the bass.

### 1.5 Data and evaluation
part 07, "experiment", part 08, "results tables 3 and 4"
- **Participants:** 28 pianists (13 professionals, 3 students, 8 teacher-performers, the 3 authors and Czerny's printed fingerings) fingered the opening RH of **7 Czerny op. 821 studies** (nos. 1, 37, 38, 54, 62, 66, 96).
- **Fragments:** 4–8 notes. The **final finger was fixed** to the consensus finger. Table 3 gives the distribution of fingerings; Table 4 gives the top-10 model fingerings with per-rule breakdowns.
- **Headline result:** in 6 of 7 pieces the most popular human fingering is in the model's top 10. Piece C's most popular fingering (32351, chosen by 10 of 28) has rank 5.
  - The exception is **Piece F**: the most popular fingering (123251, chosen by 23 of 28) ranks **13th** with total cost 25, including Large-Span = 14. It uses a constantly stretched hand position.
- **Factor analysis** (Sloboda et al.) found 5 factors covering 78% of the variance. Rules 1, 3, 10, 11 form a "stretch" factor; rules 2, 4, 5 a "position change" factor; rules 6, 7, 8 an "agility" factor. Rules 9 and 12 are separate factors.

### 1.6 Limitations the authors state
part 09, "discussion"
- The 3-note window is too short.
- The weights are ad hoc, and costs are assumed to combine linearly and independently.
- Missing rules:
  - avoid small position changes inside large ones (chunking)
  - avoid sudden hand acceleration: the hand's centre of gravity should follow a smoothed pitch contour
  - penalise sudden changes between compact and stretched spans
  - reaching a white key at the back of the keyboard while the thumb is on black
- Cognitive factors are absent:
  - standard scale and arpeggio fingerings
  - reuse of fingerings for repeated or transposed patterns ("rule of regularity", motor programmes)
  - phrase boundaries, and strong fingers on metrical accents
- Tempo is absent; faster passages should raise the weights of position-change and short-finger-on-black rules.
- Articulation is absent; staccato allows more position changes.
- Register is absent: the RH thumb is fine in the high register but awkward on isolated low white keys.
- The model is an ergonomic filter, not a model of the cognitive process.

### 1.7 Errata (from Randolph App. A)
Randolph 2023, part 10, "appendix a"
- Itemised sma and lar values in Table 4 are sometimes wrong, but their sums are correct.
- Some pcc and pcs values are wrong. Nonzero pcc with zero pcs is impossible, yet it appears.
- **Pruning was inconsistent.** The prose describes two-sided pruning (MinPrac..MaxPrac); a later passage describes MaxPrac-only pruning. The counts of playable fingerings match neither for pieces A, D, E and G. For example, Piece E listed fingerings that violate MinPrac(2,5) = 2.
- Randolph's validated reimplementation uses two-sided pruning and gives a corrected Table XXIII.
- Playable counts, original vs validated: A 2279→2035; D 1143→999; E 239→160; G 773→688.
- The paper's worked example of the Small-Span rule (E4-G4 played 3-1) is really a Large-Span case.

### 1.8 Implementation notes for a Rust deterministic engine
- **Fully implementable as specified.**
  - Store three 5×5 i8 matrices (MaxPrac, MaxComf, MaxRel) per hand, and derive the Min matrices as negated transposes.
  - Build the LH by mirroring: negate intervals and use the RH tables. This is the same reflection that Nakamura and Srivatsan use.
  - Rules are pure functions of (pitches, black/white flags, fingers) over windows of 1–3 notes.
  - Use DP/Viterbi over states (f_prev, f_cur), with transitions to f_next. The cost of the 3-note rules lands on the transition (f_prev, f_cur) → (f_cur, f_next). The state space has at most 20 states per note, so this is O(N·20·5) and trivially fast.
  - **k-best paths** (k-shortest) give ranked alternatives; Pydactyl does this and the ERR evaluation needs it.
- **Parameters:** span tables and integer weights (all 1). No learning is needed.
- **Tunable surface:** per-user hand scale factor; per-pair MaxPrac (survey data in §3.8 can calibrate it); rule weights by tempo or style.
- **Missing pieces you must add:**
  - repeated notes: Pydactyl requires the same finger; you may want a choice
  - chords (see Al Kasimi and Nakamura)
  - the LH
  - phrase segmentation and boundary constraints
  - substitutions
  - a mm-based geometry option. Jacobs 2001 replaced semitones with physical distances; that paper is not in the repo.
- **Validation data:** Parncutt Table 3 (part 08, "results tables 3 and 4") and Randolph's Table XXIII are small, exact regression tests. Each piece has a known total cost for listed fingerings, e.g. Piece A 24342313 = 6, 35453423 = 8; Piece B 3124 = 7.

---
