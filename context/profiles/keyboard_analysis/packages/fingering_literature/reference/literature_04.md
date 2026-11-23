# Literature Index: Automatic Piano Fingering, Hand Division, Span, Pedalling — part 4 of 6

Continued from `literature_03.md`.
## 4. Srivatsan & Berg-Kirkpatrick 2022, "Checklist Models for Improved Output Fluency in Piano Fingering Prediction"

**Citation.** N. Srivatsan, T. Berg-Kirkpatrick. "Checklist Models for Improved Output Fluency in Piano Fingering Prediction". *Proc. 23rd ISMIR Conf.*, Bengaluru, 2022, pp. 525–531. doi:10.5281/zenodo.7316716 (proceedings version); arXiv:2209.05622 (the stored copy). CC BY 4.0. Chunks: `sources/srivatsan_2022_checklist/chunks/01…07`.

### 4.1 Framing
part 03, "model and input representations", part 05, "experimental setup"
- **Input:** one hand of a song, as notes (p_i ∈ 1..88, onset, offset). **The hand is assumed given.**
- **Preprocessing:**
  - LH parts are reflected (pitch mirrored, finger labels as in the RH) and treated as extra RH songs.
  - **Chords** (identical onsets) are **arpeggiated from low to high**.
  - The original timings are kept for the checklist.
- **Data:** PIG, using the alternate splits of Moryossef et al. 2019 (larger train and validation sets).

### 4.2 Models

**Input representations:**
- raw pitch (88 embeddings)
- note ⊕ octave
- note ⊕ relative distance (p_i − p_{i−1}, capped at 15 semitones)
- **lattice:** horizontal white-key steps (cutoff 9) ⊕ vertical colour change

**Architecture:** embeddings feed a Bi-LSTM (2 layers, hidden size 1024, dropout 0.2, d = 256), then an MLP and a softmax over fingers. Variants:
- **Autoregressive tagger:** forward LSTM over (x, ŷ) ⊕ backward LSTM over x.
- **Prev-finger embedding:** a neural MEMM.
- **Binary checklist:** a 5-vector with one entry per finger: +1 if that finger was used within the last **100 ms** on a higher note, −1 if on a lower note, 0 otherwise.
- **Distance checklist:** per finger, the lattice distance from the note it recently played to the current note, or a learned dummy.

**Training and decoding:**
- Cross-entropy with teacher forcing; beam search with width 10.
- **REINFORCE** with a mixed objective, on 10-note chunks, with a rolling baseline over 50 examples.
- **Reward:** r = exp(−#hops − #smears)
  - **hop:** different consecutive notes with the same finger
  - **smear:** one finger on two overlapping notes
- Adam with learning rate 1e−4.

### 4.3 Fluency metrics
part 04, "learning and evaluation"; defined on output sequences
- **Thumb cross:** pitch direction opposite to finger direction, with the thumb involved.
- **Thumbless cross:** the same without the thumb; rare and hard.
- **Crossed chord:** a cross while the notes overlap in time.
- **Hop:** same finger on consecutive different notes.
- **Smear:** one finger on several notes of a chord.
- **Step spread:** mean semitones per finger step (|Δp| / |Δf|) between adjacent non-overlapping notes. For example, E→B a fifth up played 1→2 gives 7; played 1→3 it gives 3.5.
- **Chord spread:** the same for overlapping notes.
- **Anchored 4-gram precision:** the share of 4-note windows entirely correct against a single annotator.

### 4.4 Results on the PIG test set
part 06, "results", Tables 1–2. All rows are on the same test set; counts are absolute and spreads are in semitones per finger.

| Model | Thumb cross | Thumbless | Crossed chord | Hop | Smear | Step spread | Chord spread | M_gen | 4-gram |
|---|---|---|---|---|---|---|---|---|---|
| **Gold (human)** | 331 | 58 | 28 | 155 | 7 | 2.78 | 2.64 | — | — |
| Bi-LSTM (MIDI) | 228 | 84 | 47 | 485 | 148 | 2.63 | 2.56 | 66.7 | 27.8 |
| Bi-LSTM (lattice) | 217 | 43 | 40 | 334 | 74 | 2.66 | 2.62 | 69.6 | 33.1 |
| HMM-3 (Nakamura) | 220 | 32 | 31 | 196 | 84 | **2.84** | **2.72** | **70.2** | **39.5** |
| Binary checklist + REINFORCE | 261 | 49 | 26 | 217 | 42 | 2.69 | 2.64 | 69.5 | 37.1 |
| Prev-finger + REINFORCE | 283 | 53 | 54 | **53** | **12** | 2.81 | 2.76 | 68.4 | 36.5 |

- **Input representations** (Table 1, Bi-LSTM, M_gen / 4-gram):

| Representation | M_gen | 4-gram |
|---|---|---|
| raw MIDI pitch | 66.7 | 27.8 |
| note ⊕ octave | 67.0 | 28.3 |
| note ⊕ relative distance | 69.1 | 32.2 |
| **lattice** | **69.6** | **33.1** |

  M_gen here uses Srivatsan's splits, so it is not comparable to Nakamura's 64.5.
  **Caution:** the chunk's Table 1 header lists the 4-gram column before M_gen, but the values (66.7 / 27.8) match Table 2, whose header gives M_gen first. Read the larger number as M_gen.
- **The HMM** has the best M_gen and 4-gram scores, but **too many smears, too wide a spread, and too few thumb crosses**.
- **Confusion matrix** (binary checklist + REINFORCE): the thumb (89% correct) and finger 5 (83%) are most reliable; errors are mostly between adjacent fingers (3↔4, 2↔3).
- **Human evaluation:** one piano professor rated 10 validation pieces (Brahms, Mendelssohn, Rachmaninoff) on a 1–3 scale.

| Model | Comfort | Efficiency | Learnability |
|---|---|---|---|
| Bi-LSTM | 1.6 | 1.4 | 1.6 |
| Checklist + REINFORCE | 2.8 | 2.5 | 2.9 |

  The professor's comments:
  - fewer hard stretches
  - **tempo awareness matters**: the Bi-LSTM produced fingerings unplayable at Rachmaninoff speed
  - **repeated notes and chords should reuse the same fingering**
  - all models struggle with **successive chords** and with voices within a hand

### 4.5 Limitations
- Chords are linearised.
- The hand is given.
- Only hops and smears are optimised.
- Human evaluation used one rater.
- Treating music as a linear sequence of notes is inadequate for polyphony and voice-leading in one hand.

### 4.6 Implementation notes for a Rust deterministic engine
- **The fluency metrics are deterministic and cheap.** Implement them as **lint checks and post-conditions** on engine output, and compare against the gold rates normalised per note (e.g. hops / N).
- **Pianist-backed hard rules for a deterministic engine:**
  - **no smear**, except deliberate one-finger dyads such as a thumb on two adjacent white keys
  - **no thumbless crossing inside chords**
  - penalise **hops**, except on repeated notes or across rests and leaps
  - the RH/LH reflection trick
- **The binary checklist is effectively a "finger occupancy" state:** which fingers are down or recently used, and on which side of the current note. A deterministic DP can carry this as part of its state when real durations are available. This is the same idea as Nakamura's sustained-note constraint and Al Kasimi's "live notes".
- Neural models are out of scope for a deterministic crate, but the metrics and data handling transfer.

---

## 5. Al Kasimi, Nichols, Raphael, "Automatic Fingering System (AFS)"

**Citation.** A. Al Kasimi, E. Nichols, C. Raphael (Indiana University). "Automatic Fingering System (AFS)", 4 pp. © 2005 Queen Mary, University of London.
- The venue, an **ISMIR 2005** late-breaking/demo paper, is *inferred* from the copyright line and PDF metadata (`source.yaml`).
- Nakamura and Randolph also cite a follow-up: *A simple algorithm for automatic generation of polyphonic piano fingerings*, ISMIR 2007, pp. 355–356, doi:10.5281/zenodo.1415880. That paper is not in the repo.
- Chunks: Al Kasimi, Nichols and Raphael 2005, part 01, "title abstract problem application", part 02, "costs initialization dynamic programming", part 03, "trellis machine learning", part 04, "results outputs references".

### 5.1 Framing
part 01
- **Right hand. Single notes and chords.** Tempo, staccato/legato and **pedal** are listed as work-in-progress parameters but are not in the model.
- **Input:** quantized MIDI. **Output:** a text file with one finger per note. Java 1.4.
- **Hand separation:** given (RH only).
- **Personalisation:** cost curves are calibrated to the user's hand.

### 5.2 Model
part 01, part 02, part 03

**Vertical positions:** a new slice begins whenever a pitch is added or released. Each slice holds all **live** (still sounding) notes. Example (Schumann *Aufschwung*): F | F G | F C | F D | E | E G.

**Vertical cost** of a fingered chord: the sum over **adjacent finger pairs** in the chord (0–4 pairs) of C_pair(finger pair, interval).

**Horizontal cost** between successive chords: the **average** over all K'×K pairs (one note from each chord) of the pair transition cost.
- If finger order matches pitch order, the transition is costed like the vertical function (as if the notes were simultaneous).
- If it does not, it is a **crossover**:
  - thumb crossings have **low cost**
  - non-thumb crossings are **heavily penalised**

**Per-finger-pair cost curve** (Fig. 4, pair 2-5), cost on a 0–10 scale against distance in half-steps:

| Distance | 1 | 2 | 3 | 4–7 | 8 | 9 | 10 | 11–15 | >15 |
|---|---|---|---|---|---|---|---|---|---|
| Cost | 10 | ≈6.7 | ≈3.3 | 0 | 4 | 8 | ≈9 | 10 | 10 |

- The general shape is the same for every pair: high cost when cramped, falling to a comfort zone of 0, then rising to a cap of 10.
- Three breakpoints depend on the pair and are set by asking the user:
  - smallest easy interval (4 for 2-5)
  - comfort zone (4–7)
  - rising difficulty (7–11)
  - "nearly impossible" beyond 11
- The prose names the breakpoints as 3, 7 and 9, which does not quite match the plotted points. Use the calibration semantics, not exact numbers.

**Trellis and dynamic programming:**
- Each layer is one chord. Nodes are finger subsets of size K: C(5, K), with **finger order equal to pitch order**. Two-note chords have 10 nodes, three-note chords 10 nodes.
- Layers are fully connected.
- Start→node costs are the node's vertical cost; last layer→end costs 0.
- An edge costs the vertical cost of its target plus the horizontal cost.
- Search with **Dijkstra**, equivalent to Viterbi on the DAG.
- **[derived]** Complexity is O(Σ_t |L_{t−1}|·|L_t|·K_{t−1}·K_t) with |L| ≤ 10, so it is linear in length.

### 5.3 Learning
part 03, "trellis machine learning"; in progress, no results
- Training data: MIDI plus fingering files. Error is the number of mismatched notes.
- An iterative, simulated-annealing-like **local credit assignment**:
  - For each misfingered note, collect the vertical pairs (adjacent notes in its chord) and the horizontal pairs (with the nearest one or two notes in neighbouring chords).
  - **Skip pairs whose distance is in the comfort zone.**
  - Lower the cost of the target fingering's pairs and raise the cost of the predicted fingering's pairs.
  - Repeat until the error falls below a threshold or an epoch limit is reached.

### 5.4 Results and limitations
part 04, "results outputs references"
- **Qualitative only.** The outputs are Ellmenreich's *Spinning Song* and Chopin's op. 25 no. 7, with the full digit sequences transcribed in the chunk.
- **Stated failure:** on the A-C-F and B♭-C-E chords the model chose 1-3-5 and 2-3-4 instead of the natural 1-2-5 and 1-2-4. The 1-2-4 choice needs the **same finger (1) on adjacent notes B♭→C**, which the model slightly penalised. Smooth horizontal 1→2 and 5→4 transitions outweighed an awkward vertical stretch.
- The authors conclude that the **balance between horizontal and vertical weights** is the key tuning problem.
- No quantitative evaluation. Nakamura's chord-HMM version scored M_gen 61.2.

### 5.5 Implementation notes for a Rust deterministic engine
- **The cleanest deterministic polyphonic architecture of the five.** Use a chord-slice trellis over finger subsets in pitch order, with live-note persistence: a held note keeps its finger across slices.
- **Parameters:** per-pair piecewise-linear curves (4 breakpoints and a cap), derivable from Parncutt's MinPrac, MinComf, MaxComf and MaxPrac, plus crossover penalties.
  - **[derived]** Map Parncutt to Al Kasimi like this: cost 10 at or beyond MaxPrac and MinPrac, 0 between MinRel and MaxRel, linear in between. Vertical cost then comes from Parncutt's MinPrac..MaxPrac for simultaneous notes. Parncutt's tables are for consecutive legato notes, and simultaneous chord spans are roughly the same practical limits.
- Use **max or sum, not average, for horizontal cost**. Averaging dilutes a single bad finger move in a big chord (cf. Nakamura's K^−ζ).
- **Add explicit options:**
  - same finger on adjacent keys: a thumb sliding or covering two white keys, or the same finger on a repeated note
  - weak-finger and black-key rules from Parncutt
- Learning can be offline; ship the resulting constants.

---
