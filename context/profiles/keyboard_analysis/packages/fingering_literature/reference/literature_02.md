# Literature Index: Automatic Piano Fingering, Hand Division, Span, Pedalling — part 2 of 6

Continued from `literature_01.md`.
## 2. Nakamura, Saito, Yoshii 2020, "Statistical learning and estimation of piano fingering"

**Citation.** E. Nakamura, Y. Saito, K. Yoshii. *Information Sciences* 517:68–85 (2020). doi:10.1016/j.ins.2019.12.068 (preprint arXiv:1904.10237). Chunks: Nakamura, Saito and Yoshii 2020.

### 2.1 Problem framing
part 04, "hmm model" §5.1
- **Input:** a MIDI-like sequence (p_n, t_n, t̄_n), ordered by onset. **The LH and RH parts are assumed separately given.** For automatic hand separation the paper points to its ref. [23], the merged-output HMM (Nakamura, Ono, Sagayama, ISMIR 2014).
- **Output:** a finger f_n ∈ {1..5} per note.
- **Polyphony:** handled at note level by onset ordering, with an optional chord constraint and a separate chord HMM.
- **Timing:** temporal distances are *not* modelled, except the 30 ms test for chord co-membership.
- In testing, **staff = hand**: upper staff is RH, lower staff is LH (part 06, "evaluation setup and measures").

### 2.2 PIG dataset
part 02, "review and dataset", part 03, "individual difference"
- **Content:** 150 pieces by 24 composers, each a section of about 1 page, about 20 bars and about 300 notes.
- **Fingerers:** pianists who were conservatory graduates or had played for more than 20 years. They gave the fingering they actually used in performance.
- **Subsets:**

| Subset | Pieces | Fingerers per piece | Annotated instances | Notes |
|---|---|---|---|---|
| Bach | 10 | ≥4 | 40 | 14,628 |
| Mozart | 10 | 6 | 60 | 15,276 |
| Chopin | 10 | 5 | 50 | 20,110 |
| Miscellaneous (training) | 120 | 1–2 | 159 | 50,030 |
| **All** | **150** | | **309** | **100,044** (48,726 unique) |

- **File format:** one line per note in onset order: `n t_on t_off p v_on v_off c f`
  - note ID; onset and offset in seconds; pitch; onset and offset velocities
  - `c` = hand (0 = RH, 1 = LH)
  - `f` = finger, 1..5, **negative for the LH**
  - substitutions are written `1_2`
  - Each piece also comes with a PDF score.
- **Download URLs:** `http://beam.kisarazu.ac.jp/research/PianoFingeringDataset/`; Randolph cites `https://beam.kisarazu.ac.jp/~saito/research/PianoFingeringDataset`. Check the licence before shipping any derived tables.
- **Substitutions** occur on **0.34%** of notes. The models use the first finger.
- **Randolph's critique** (Randolph 2023, part 05, "ch4 corpus development" §4.4):
  - The PIG data look synthetic or humanized, with identical chord onsets and uneven velocity humanization.
  - There is no secondary sort order within chords.
  - The data are separated by hand rather than by staff.
- **Inter-annotator agreement** (Fig. 1):
  - Mean match between 2 players is about 0.62 (Bach), about 0.71 (Mozart) and about 0.79 (Chopin).
  - Agreement falls as a power law in the number of players: y = x^−0.689, x^−0.468, x^−0.315 respectively.
- **Finger choices per note (Fig. 2a):**

| Subset | 1 choice | 2 choices | 3 choices | 4 choices |
|---|---|---|---|---|
| Bach | 39.3% | 49.4% | 10.9% | 0.4% |
| Mozart | 45.6% | 40.9% | 12.9% | 0.5% |
| Chopin | 62.1% | 33.1% | 4.5% | 0.3% |

  Denser, more chordal music leaves fewer choices. For example, the lower note of an RH octave is almost always the thumb.

### 2.3 Models
part 04, "hmm model", part 05, "higher order chord hmm dnn"

**First-order HMM** (after Yonebayashi 2007 and Nakamura 2014):
- P(f) = P(f_1) ∏ P(f_n | f_{n−1})
- P(p | f) = P(p_1 | f_1) ∏ P(p_n | p_{n−1}, f_{n−1}, f_n)
- Decoding is **Viterbi**.

**Output-probability structure:**
- **Transposition symmetry:** P depends only on the interval, F(p_n − p_{n−1}; f_{n−1}, f_n). This reduces about 88²·5² ≈ 2·10⁵ parameters.
- **Pitch representations:**
  - integral semitones
  - **2D lattice:** x counts white-key steps, with a black key counting as half-way. y ∈ {0, ±1} records a change of key colour.
- **Leap cutoff** δ_p^max = **15**: larger leaps are pooled.
- **Optional time-inversion symmetry:** F(d; f, g) = F(−d; g, f).
- **Optional LH↔RH reflection symmetry:** F^L(d) = F^R(−d_x).
- With the full training data, *not* imposing these extra symmetries is slightly better; reflection costs more than time inversion. With small data, imposing them helps (Fig. 7, part 07, "accuracy comparisons").

**Chord constraint:** consecutive notes with |t_n − t_{n−1}| ≤ **30 ms** are in the same chord. Within a chord, the output probability is 0 if pitch direction and finger direction disagree (no crossing).

**Higher-order HMM** (order m):
- **Output factorisation:** P(p_n | p_{n−m..n−1}, f_{n−m..n}) ∝ ∏_{l=1..m} P(p_n | p_{n−l}, f_{n−l}, f_n)^{α_l}
- **Transition smoothing:** P(f_n | f_{n−m..n−1}) = (1 − Σλ) P_ML(order m) + Σ_{l=1..m−1} λ_l P_ML(order l)
- **Learned weights** (Bayesian optimisation, GP-MI, 200 iterations; Table 3):

| Model | Weights |
|---|---|
| 1st order | α1 = 0.964 |
| 2nd order | α1 = 0.556, α2 = 0.407, λ1 = 0.474 |
| 3rd order | α1 = 0.448, α2 = 0.292, α3 = 0.194, λ1 = 0.470, λ2 = 0.504 |

**Chord HMM** (a probabilistic version of Al Kasimi's model):
- **States** are finger assignments for whole "chords": note groups whose onsets lie within 30 ms.
- **Notes still sounding** are included in the following chords, and their finger is fixed.
- **State count:** fingerings without crossings and with distinct fingers number C(5, K) for K pitches. The print garbles this as "K5".
- **Vertical cost:** V(Q, G) = Σ_{k≠k'} v(Q_k', Q_k; G_k', G_k)
- **Horizontal cost:** H = Σ_{k'} Σ_k h(Q_k', Q_k; G_k', G_k)
- **In probabilistic form:**
  - transitions use exponents β1 (across chords) and β2 (within a chord)
  - outputs use exponents γ1 and γ2
  - a per-chord weight K^−ζ counteracts the bias from chord size
- Fitted values: β1 = 0.94, β2 = 4.70, γ1 = 7.53, γ2 = 5.29, ζ = 0.10.

**DNN baselines:**
- Feed-forward network with 3 hidden layers × 32 units, and a 3-layer LSTM × 32.
- Input is integer pitch; window ±5; trained per hand with Adam, batch size 3.

**Complexity [derived]:** Viterbi for an order-m HMM is O(N·5^{m+1}). The chord HMM is O(Σ |S_t|·|S_{t−1}|·K·K'), with |S| ≤ C(5, ⌈5/2⌉) = 10 per hand.

### 2.4 Evaluation measures
part 06, "evaluation setup and measures", part 09, "appendices and references" (App. A)
- **M_gen:** mean match rate over all ground truths.
- **M_high:** best match against any single ground truth.
- **M_soft:** a note counts as correct if any ground truth agrees.
- **M_rec (recombination):**
  - Edit-distance DP over a ground-truth selector z_n.
  - Switching ground truth costs **C_rec = 1**, allowed only where the fingers match. The generalisation C'_rec applies where they do not, with ∞ meaning disallowed.
  - A substitution costs **C_sub = 1**.
  - M_rec = (N − E_rec) / N.
- Code: `https://statpianofingering.github.io/demo.html`.

### 2.5 Results
Training uses the 120 miscellaneous pieces; testing uses 30 pieces by Bach, Mozart and Chopin (part 06, part 07, "accuracy comparisons"). Table 2, in %:

| Method | M_gen | M_high | M_soft | M_rec |
|---|---|---|---|---|
| 1st HMM | 61.7 | 68.3 | 82.8 | 74.0 |
| 2nd HMM | 64.3 | 70.8 | 85.3 | 77.6 |
| **3rd HMM** | **64.5** | **71.0** | **85.5** | **77.8** |
| Chord HMM | 61.2 | 67.7 | 81.7 | 73.8 |
| FF DNN | 61.5 | 66.2 | 82.5 | 69.5 |
| LSTM | 61.3 | 66.1 | 82.8 | 69.5 |
| Human (leave-one-out) | 71.4 | 79.1 | 90.8 | 84.3 |

- **Balliauw VNS**, the representative constraint-based method: it parsed only 14 of the 30 test pieces. On those 14 (Table 4) its scores were M_gen 56.7, M_high 62.4, M_soft 78.2, M_rec 67.5, against 60.5, 67.9, 82.4, 73.7 for the 1st-order HMM.
- **Architecture ablation** (Table 5, 1st order): **the lattice representation beats the integral one, and the chord constraint helps.** M_gen is 61.6 with lattice and constraint, against 59.5 with integral and no constraint.
- **Learning curves:** A(N) = 63.9 − 524.6/√N (1st order) and 66.3 − 513.3/√N (2nd order). Even infinite data stays below human agreement of about 71%.

### 2.6 Error analysis and limitations
part 08, "example results and discussion"
- **Phrase boundaries and rests** are ignored. Across a rest the human uses 5-5 where the HMM picks 5-1.
- **Articulation** changes which transition matters.
- The HMMs use a **narrower finger range** than humans for isolated thirds: 1-2 or 2-3 instead of 1-3 or 2-4. Fifths are not always given 1-5.
- **Apex notes** are not reliably given 1 or 5.
- **Two-hand interdependence** is not modelled. When the hands overlap, humans use smaller ranges.
- **Very long-range melodic dependence** through intervening chord notes is missed.
- **Errors in hand assignment** propagate into fingering errors.
- The authors argue that ad-hoc rules are not a fundamental fix.
- **Future work:** temporal features (IOI, duration, gaps); phrase, voice and apex structure; individuality and hand size; slurs; velocities; **pedalling** ("not studied here"); and the cost-vs-probability distinction (high-probability ≠ low-cost).

### 2.7 Implementation notes for a Rust deterministic engine
- **The HMM is deterministic at inference.** With trained tables shipped as constants (u16 or f32 log-probabilities), Viterbi gives a reproducible answer.
- **Table sizes:**
  - 1st order: transitions 5×5; outputs 5×5×(lattice dx in −15..15 × dy in −1..1), a few thousand floats.
  - 3rd order: transitions 5⁴ plus smoothing; outputs 3 pairwise tables plus 3 α values.
- **Learned parameters are required.** They need the PIG data: check the licence, or retrain on your own corpus. Training is just counting: maximum-likelihood normalised counts, plus a small grid or Bayesian search for α and λ.
- **Directly reusable without learning:**
  - the lattice pitch representation (key geometry)
  - the 30 ms chord rule
  - no-crossing within chords
  - the leap cap of 15
  - the RH/LH reflection
  - the metrics M_gen, M_high, M_soft, M_rec (implement M_rec as a small DP)
- **A hybrid fits a deterministic crate well:** log-probability = −cost. Parncutt or Al Kasimi costs can serve as priors or features, with Viterbi as the shared engine.
- The chord HMM's sustained-note constraint (a held note keeps its finger) is a useful **hard constraint** when the input has real durations. The authors suspect it hurt them only because the PIG durations are synthetic.

---
