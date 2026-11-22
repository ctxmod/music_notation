# Literature Index: Automatic Piano Fingering, Hand Division, Span, Pedalling — part 6 of 6

Continued from `literature_05.md`.
## 9. Design notes for the architect (synthesis, [derived])
- **Core engine.** Build one Viterbi/DP framework, one per hand, whose state is the finger assignment of the current chord slice (C(5,K) states, ordered) plus the fingers held by live notes.
  - Edge cost = Σ weighted rule costs: Parncutt rules 1–12 generalised to chord slices, Al Kasimi's vertical cost, and the no-crossing-in-chord hard constraint.
  - Emit **k-best** paths.
  - This covers Parncutt (K = 1), Al Kasimi (K ≥ 1) and Nakamura (replace costs with −log-probabilities) in one deterministic machine.
- **Geometry.** Use the lattice (white-key index plus colour) and optionally millimetres (13.7 mm per semitone average; a per-key physical table is better). Hand-size scaling should default to "medium = 9th practical reach", calibrated with Randolph's survey.
- **Hard constraints:**
  - spans within MinPrac..MaxPrac, consecutive and simultaneous
  - finger order equals pitch order within a slice
  - a held note keeps its finger (with optional substitution moves)
  - at most one finger per key, except an explicit one-finger dyad (thumb)
- **Soft costs:**
  - Parncutt's 12 rules with integer weights
  - tempo-dependent weights (Parncutt §9)
  - a repeated-note policy
  - a phrase or rest boundary, which resets the position-change cost
- **Hand separation stage, before fingering:**
  - default to staff or voice
  - a cost-based reassignment search for over-span slices
  - flag every redistribution
  - Nakamura 2014 is the model to study
- **Pedal stage, after fingering:**
  - harmony-change detection gives syncopated damper changes (press after onset, release at the next change)
  - add pedal where fingered sustain is impossible (span over MaxPrac, or a held bass under hand movement)
  - avoid low close-position blur
  - output in abcDF-like or MIDI CC64 form
  - also allow the reverse: assume pedal and relax legato constraints on fingering
- **Validation:** exact Parncutt cost regressions; Beringer standard fingerings; PIG match rates if licensable; Srivatsan fluency lint; ERR for k-best lists.
