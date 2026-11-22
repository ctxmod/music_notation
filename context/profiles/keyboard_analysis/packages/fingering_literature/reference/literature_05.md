# Literature Index: Automatic Piano Fingering, Hand Division, Span, Pedalling — part 5 of 6

Continued from `literature_04.md`.
## 6. Cross-source synthesis

### 6.1 Comparison table

| | Parncutt 1997 | Al Kasimi 2005 | Nakamura 2020 | Srivatsan 2022 | Randolph 2023 (CRF) |
|---|---|---|---|---|---|
| **Approach** | Expert rule cost (12 rules); exhaustive or DP | Expert cost (vertical + horizontal) over a chord trellis; Dijkstra; optional learning | Generative HMM orders 1–3 plus chord HMM; Viterbi | Discriminative Bi-LSTM with checklist; beam search; REINFORCE | Discriminative linear-chain CRF; Viterbi |
| **Chords** | No (melody only) | **Yes, native** (chord states, live notes) | Note-level with a 30 ms no-crossing constraint; chord HMM native | Arpeggiated low→high; smear and crossed-chord metrics | 30 ms concurrency features; total order; Pydactyl baselines arpeggiate |
| **Hand separation** | Assumed (RH only) | Assumed (RH only) | Assumed given (staff = hand); refers to merged-output HMM [23] | Assumed given | Assumed (staff); critiques PIG's hand-based channels |
| **Determinism** | Fully deterministic (ties broken arbitrarily) | Deterministic at inference | Deterministic at inference once trained | Beam search over a neural net: deterministic given fixed weights, but heavy | Deterministic at inference |
| **Data needs** | None (hand-set tables) | User calibration; optional training | PIG-scale annotated data (≈50k notes for training) | PIG; GPU | PIG plus own corpora |
| **Accuracy** | 6 of 7 Czerny "most popular" fingerings in top 10; via VNS ≈ M_gen 56.7 | Qualitative only; as chord HMM M_gen 61.2 | **M_gen 64.5 / M_high 71.0 / M_soft 85.5 / M_rec 77.8** (3rd order) | M_gen ≈ 69.5 on other splits (HMM 70.2 there) | M_gen 64.6 vs HMM 64.5 (n.s.) |
| **Pedal** | No (mentions pedal legato in hard passages) | Listed as future work; not modelled | No ("not studied") | No | No; abcDF can annotate it |

Human agreement is the ceiling: M_gen about 71, M_high about 79 (Nakamura).

### 6.2 Consensus physical constraint model
These are the facts all sources share, to be used as hard and soft constraints.

1. **Finger order on the keyboard.**
   - In the RH, fingers 1→5 lie low→high; mirror this for the LH.
   - **Within a chord the finger order must match the pitch order.** This is a hard constraint for Al Kasimi, Nakamura and Randolph (ERR §5.5), and Srivatsan's crossed-chord counts confirm it is rare (gold: 28).
   - **Non-thumb crossings** in melody are rare and costly: Parncutt forbids them, Al Kasimi penalises them heavily, Srivatsan's gold has 58. They occur historically (Couperin, Bach; Chopin and Busoni over-passing; Walker/Chopin).
2. **Finger-pair spans** (RH, semitones, consecutive legato; Parncutt Table 1).
   - The hard limit is MinPrac..MaxPrac and the zero-cost zone is MinRel..MaxRel.
   - **Thumb pairs have huge ranges:** 1-5 runs −1..15, 1-2 −5..10.
   - **Non-thumb adjacent pairs are tiny:** 3-4 is 1..4 and 4-5 is 1..5.
   - Physical scale: about 13.7 mm per semitone. Scale by about 0.9 for small hands.
   - **Independent checks:**
     - Al Kasimi's 2-5 comfort zone of 4–7 half-steps with a limit near 11, against Parncutt's 2-5 Rel 5–6, Comf to 8 and Prac 2–10.
     - Randolph's survey: the median RH practical 1–5 reach is a 9th (14 semitones), with 10ths common. Parncutt's MaxPrac(1,5) is 15.
     - Cortot's virtuoso targets: 1-5 = 17, 3-4 = 7.
     - Kullak (Beethoven edition) thinks a tenth should be practicable for medium hands (study library, part 04, "concerto performance rules", fn. ch2-3).
3. **Black and white keys.**
   - **Avoid 1 and 5 on black** in single-note passages, especially when the neighbouring notes are white (Parncutt rules 10–11; Taub; Hofmann on scales on black keys).
   - **Good thumb passes are non-thumb on black → thumb on white.** This is why scale fingerings put the thumb after black keys (Parncutt rule 12; Chopin starting pupils on B major).
   - The **thumb on black is acceptable in chords, octaves (use 1-4 on black-key octaves; Taub), leaps and accents** with arm support (Arrau, Chopin). Modern pedagogy relaxes this, so weight the rule down for chordal or virtuoso textures.
   - Model key geometry with a **lattice** (white-key steps plus a colour flag), which beat semitones in both Nakamura (Table 5) and Srivatsan (Table 1). Hart's model conditions on the 4 key-colour transitions.
4. **Thumb crossing.**
   - It is the **only** routine way to change hand position in legato: 1 under 2/3/4, or 2/3/4 over 1.
   - The maximum pass is MinPrac(1,g) = −5, −4, −3, −1 for g = 2, 3, 4, 5.
   - Arpeggios exceed this, so they are not finger-legato in fast tempo (Parncutt).
   - Pianist gold data have about 331 thumb crosses in the PIG test set; models under-produce them (HMM 220).
   - Hofmann: move the thumb early, toward its key as soon as the previous key is released. Arrau: pass by forearm rotation, with finger 4 "pole-vaulting".
5. **Weak fingers 3-4-5.**
   - 4 and 5 are weaker (Parncutt rule 6).
   - Runs of 3-4-5 are costly (rule 7).
   - **3→4 is harder than 4→3** (rule 8; 1663 vs 1142 occurrences).
   - 4 on black after 3 on white is costly (rule 9).
   - Arrau avoided 3-4-5 in diatonic passages. Chopin called 3 and 4 "Siamese twins". Hofmann says the thumb and finger 3 need the most control, and that 4 and 5 should be trained loose.
   - Most model confusions are 3↔4 and 2↔3 (Srivatsan), and Randolph's AdjLong treats these as half-errors. These pairs are partly interchangeable, so tie-breaking between them is a style choice.
6. **Position changes and economy of movement.**
   - Minimise the number and size of hand shifts (Parncutt rules 4–5), going back to Diruta, Sancta Maria and C. P. E. Bach.
   - Change position at **phrase boundaries and rests**: Nakamura's errors, Randolph's segmentation, and Parncutt/Clarke (changes are more likely at phrase boundaries).
   - Across rests and large leaps the sequential constraint largely disappears (Srivatsan caps steps at 15; Nakamura and Randolph cap dx at 15).
7. **Apex and extreme notes:** the top note of a run is usually 5 (RH) or 1, and the lower note of an RH octave is almost always the thumb (Nakamura).

### 6.3 Polyphony
- **Parncutt:** none.
- **Al Kasimi:** chord-slice states built from live notes; vertical cost over adjacent finger pairs; horizontal cost averaged over all cross pairs.
- **Nakamura:**
  - note-level HMM plus a 30 ms chord detector and no-crossing constraint (+1.6 M_gen)
  - chord HMM with sustained-note finger persistence, which scored below the 1st-order HMM, possibly because PIG durations are synthetic
  - Chopin's chord-dense music gives the *highest* human agreement
- **Srivatsan:** arpeggiate; smear and crossed-chord metrics; still weak on successive chords and voice-leading within a hand.
- **Randolph:** concurrency and overlap features, and the observation that piano chord fingering is highly constrained (ordered). Pydactyl baselines arpeggiate.
- **Gap:** none of the five handles **voices within one hand**: a held note under a moving voice, substitution to free a finger, or finger legato of one voice while another is detached. Al Kasimi's live notes and Nakamura's sustained constraint are the starting points.

### 6.4 Hand assignment and separation
- **Every core model assumes it is given.** Staff = hand in Nakamura's testing, Randolph's CRF and Pydactyl. PIG's `c` field records the hand as played.
- **Nakamura 2014 merged-output HMM** (ref. [23], ISMIR 2014, pp. 531–536; not in the repo) estimates the hand parts, with fingering, from a mixed MIDI stream. **It is the main reference to acquire for hand separation.**
- **Randolph** names the "integrated" (cross-staff) mature axis. The abcDF `<`/`>` hand markers encode it. Match rates are invalid when the hand assignment is debatable.
- **Two-hand interdependence:** when the hands overlap, the fingerings use smaller ranges (Nakamura error analysis). See Parncutt, Sloboda, Clarke 1999 in the bibliography.
- **Pedagogical constraint** (Taub, study library, part 04, "chapter 2 hands and feet"):
  - Redistributing hands changes musical character. Beethoven explicitly marked the RH helping the LH only once (Op. 2 no. 3, I, mm. 129–137), and Taub opposes other redistributions.
  - A hand-separation engine should **default to the notated staff or voice**, and offer redistribution only as an optional "ease" transformation, flagged as such.
- **Engine heuristics derivable from the sources [derived]:**
  1. The default hand is the staff.
  2. Reassign a note only when a single hand's span within a chord-slice exceeds MaxPrac(1,5) for that hand while the other hand has a free finger within reach.
  3. Use Parncutt and Al Kasimi costs to compare the two assignments.
  4. Check the inter-hand crossing and overlap constraint.

### 6.5 Pedal
**Say it plainly: none of the five core models uses, predicts or outputs pedalling.**
- Parncutt mentions pedal legato only for impossible thumb-on-black passes.
- Al Kasimi lists pedal as future work.
- Nakamura lists pedalling as unstudied.
- Srivatsan omits it.
- Randolph declares the field unpedalled. He supplies only the **abcDF damper and soft pedal annotation** and Dactylize pedal capture. His CRF overlap features explicitly *assume no damper*.

The pedagogical sources give rules a deterministic pedal engine could encode (details in §7):
- **Legato ("syncopated") pedal:** press the damper **just after** the new note is struck, and release it as or just before the next harmony is struck. Never press it simultaneously with the stroke (Hofmann; Schmitt ch. 1).
- **Change the pedal with each change of harmony.** Melody notes in the treble may each need a separate change (Schmitt ch. 2–3; Hofmann).
- **The pedal frees the fingers.** Keys can be released early (staccato) while the sound continues, which simplifies fingering: chord sequences can then use "the simple fingering of the common chord" (Schmitt ch. 1). This is the main **fingering↔pedal interaction**.
- **Wide arpeggiated chords:** press the pedal at the *start* of the arpeggio so that all notes sustain (Schmitt ch. 1).
- **Register and texture:** low close-position chords blur; dissonance is more tolerable when passing tones lie more than an octave above the bass (Hofmann). Bach should be pedalled sparingly. Use the pedal to hold harmony beyond the hand's reach (Hofmann).
- **Una corda** is for tone colour, not loudness (Hofmann, Taub). **Sostenuto** sustains only notes already held (Taub, Shockley).

### 6.6 Legato and repeated notes
- **Legato.**
  - Parncutt assumes **finger legato** throughout. MinPrac for the thumb is set so arpeggios are not legato.
  - Hofmann's legato rule: the finger holds its key until the next tone is heard, and two fingers are always down in fast passages.
  - Chopin and organists sustain legato by **substitution** and by sliding a finger from a black key to a white one, rather than relying on the pedal (Walker).
  - None of the models output substitutions. PIG has them on 0.34% of notes; abcDF can encode them.
- **Repeated notes.**
  - Parncutt forbids the same finger on consecutive notes, including repeated pitches.
  - Pydactyl's modification requires the **same** finger on repeated pitches.
  - Al Kasimi slightly penalises the same finger on adjacent *different* notes, which caused an unnatural chord fingering.
  - Srivatsan's pianist wanted repeated notes and chords to reuse their fingering, while hops (same finger on different notes) are penalised.
  - Hofmann and Taub: fast repetitions use **changing fingers** (e.g. 3-2-1) with a wiping motion, while portato uses the same finger for uniformity.
  - **Deterministic rule [derived]:**
    - repeated pitch, slow or portato tempo: the same finger is allowed and preferred
    - fast repetition (inter-onset interval below about 150 ms; threshold to be chosen): use a finger-change pattern
    - different pitch with the same finger: penalise unless separated by a rest or staccato, or by a gap of 60 ms or more (Randolph's separation threshold)
- **Articulation and tempo** change the optimum (Parncutt §9; Randolph's CRF features; Srivatsan's evaluator). Staccato allows hand shifts and same-finger moves. Fast tempo raises the weights on position changes and black-key penalties.

### 6.7 Evaluation stack the crate should implement
- **Per-note:** accuracy and the M_gen, M_high, M_soft and M_rec match rates (Nakamura).
- **List-level:** ERR with the Hamming, AdjLong and trigram distances (Randolph).
- **Fluency lint:** the counts and spreads from Srivatsan, compared with the gold rates.
- **Regression corpora:**
  - Parncutt Table 3/4 and Randolph's validated Table XXIII (exact costs)
  - the Beringer scales, arpeggios and broken chords (exact standard fingerings)
  - the PIG test set (30 pieces) if its licence allows

### 6.8 Open questions and gaps
1. **Rule weights and nonlinearity.** Parncutt's equal weights are untuned. A learnable or derived weighting is needed, e.g. fitting integer weights to PIG or Beringer by a simple search, then freezing them.
2. **Chord-to-chord transitions and voices within one hand** are weak in every system.
3. **Hand separation:** there is no model in the repo. Acquire Nakamura 2014 (merged-output HMM). Two-hand interdependence is unmodelled.
4. **Pedalling:** no computational model at all. Rules come from treatises (§7). The coupling between fingering and pedal is unexplored.
5. **Substitution and finger legato in polyphony:** no model.
6. **Physical geometry:** semitones versus millimetres, key width, the depth of black versus white keys, and the angle of the hand with register (Parncutt), and Jacobs' refinements.
7. **Tempo and articulation dependence:** only the CRF and the checklist use timing.
8. **Cognitive regularity:** the same fingering for repeated or transposed patterns, and standard scale fingerings. A deterministic engine can do this with pattern matching and caching. No paper models it explicitly.
9. **Multiple ground truths:** produce **k-best diverse** suggestions rather than a single answer (Randolph).
10. **Data quality:** PIG is humanized/synthetic MIDI. Durations are unreliable for sustained-note constraints.

---

## 7. Study-library passages relevant to fingering, span, hand use, pedalling and legato

### 7.1 Core pedagogical sources
**study library** lists the 4 sources below.

**Schmitt, *The Pedals of the Pianoforte* (1893)** — study library. The best source for a pedal rule set. OCR is noisy and the music examples are garbled.
- part 03, "chapter one", pp. 9–20:
  - Pedal timing relative to fingers: the "rests" of finger and foot must never coincide.
  - Notation of pedal as notes and rests on a separate line.
  - Legato pedalling: the foot moves just *after* the fingers.
  - The pedal lets chord passages use simple fingerings.
  - Press the pedal at the start of wide arpeggiated chords.
  - Fortissimo passages played staccato with pedal.
  - Accenting one tone of a chord through the shape of the finger.
- part 04, "chapter two", pp. 38–49: sympathetic resonance and overtones. Each melody note gets its own pedal movement, except short notes. Pedal more often in slow tempo or ritardando than in fast.
- part 05, "chapter three", pp. 53–70:
  - Hold the pedal through one harmony and **change it at each new harmony, slightly late**.
  - Low close positions sound harsh.
  - Pedal in scales and passages over a sustained strong bass.
  - Effects of the dynamic balance.
- part 06, "chapter four", pp. 86–103:
  - Composers' pedal marks are unreliable.
  - Critique of "sempre pedale".
  - **A summary list of when to pedal:**
    - long notes
    - the longer notes within a melody
    - short notes separated by rests
    - to strengthen the touch
    - for pp
    - for echoes
  - Broken chords; scales with pedal only in presto with sustained tones.

**Hofmann, *Piano Playing* and *Piano Questions Answered*** — study library
- part 05, "the use of the pedal":
  - The ear decides.
  - Press the pedal with a quick, full motion **immediately after the key stroke, never with it**.
  - Harmonic clarity is the baseline, but blending is acceptable when foreign tones lie more than an octave above the bass.
  - Use the pedal for colour; do not use it to cover weak fingers or as a substitute for forte.
  - The una corda changes quality, not just quantity.
- part 12, "the pedals" (Q&A):
  - The general rule as above.
  - Release with the next tone and re-press; change with the harmony or with passing notes.
  - The pedal helps where the hand's reach cannot hold a harmony.
  - Bach: pedal used qualitatively, but sparingly in amount.
  - Both pedals may be used at once.
- part 10, "technique" (Q&A sections):
  - **§7 Stretching:** caution; Schumann's injury.
  - **§8 Thumb:** move it early in scales; keep it near finger 2.
  - **§9–10:** 4th and 5th fingers, weak fingers.
  - **§12 Legato:** hold until the next tone is heard; two fingers down in fast passages.
  - **§15 Fingering:**
    - C-scale fingering is wrong for black-key scales.
    - **Chromatic scale: RH thumb always on E and B, LH thumb on F and C**, with 3–4 consecutive fingers in between.
    - Mordents use whatever fingers are already in place.
  - **§17 Octaves:** wrist versus arm.
  - **§18 Repeated notes:** a wiping finger motion.
  - **§19 Double thirds:** double legato.
- part 04, "correct touch and technic": touch and legato basics.

**von Arx, *Claudio Arrau's Piano Lessons*** — study library
- part 07, "chapter five the arrau technique", §5.9 "Fingering and Use of the Thumb" (from about line 1183):
  - Omit finger 4 and **avoid 3-4-5** in diatonic passages.
  - Use the **thumb on accents, to relieve extensions, and on black keys in leaps** (Waldstein finale, Appassionata, *Chiarina*).
  - Hand off sustain to the pedal when the span is impractical.
  - Pass the thumb by forearm rotation.
  - The thumb-on-black "taboo" is relaxed with arm movement.
- Earlier sections cover hand and wrist position (§5.1), fingers (§5.2) and rotation (§5.7).
- Marginal: part 05, "chapter three rudolf breithaupt" (weight technique), and lesson appendices part 11, "appendix 1 2", part 20, "appendix 3 3", part 26, "appendix 5 bennett lerner", part 27, "appendix 6 bennett lerner 1967", with scattered fingering remarks.

**Partington, *Making Music*** — study library. Performance psychology; fingering mentioned only in passing (part 03, part 04). **Marginal.**

### 7.2 Beethoven package
**Taub, *Playing the Beethoven Piano Sonatas*, ch. 2 "Beethoven, His Hands, and His Feet"** — study library, part 04, "chapter 2 hands and feet". **Highly relevant.**
- Classical principles: no thumb on black keys in single-note passages; **1-4 for black-key octaves**.
- Articulation determines fingering: portato uses the same finger; slurred pairs 3-2; Beethoven's 1-3-1-3 for legato.
- Trill fingerings (1-3, 2-3; 4-5, 3-4 or 3-5 while 1-2 hold a melody, as in Op. 111). The choice depends on the piano's aftertouch.
- Beethoven's own fingerings exist for musical reasons, e.g. Op. 2/2 one-hand passages and Op. 53 octave glissandos.
- **Hand redistribution:** only Op. 2/3 mm. 129–137 is marked by Beethoven; Taub argues against redistributing.
- Pedals:
  - una corda on/off
  - tre corde markings
  - sostenuto not for Beethoven
  - damper depth (half-pedal)
  - long blurred pedals (Op. 31/2 recitatives, Op. 27/2 "senza sordini")
  - the Op. 53 hand crossing
- The programme chapters part 11, "program 1"–part 19, "program 9" contain piece-specific pedal and fingering notes. **Marginal to moderate.**

**Kullak, *Beethoven's Piano-Playing*** — study library
- part 03, "general remarks to 1809": Beethoven's legato versus Mozart's "choppy" style; Beethoven's heavy pedal use; Wölffl's tenth-span hand.
- part 04, "concerto performance rules": slurs as general legato signs; fn. ch2-3 says a tenth should be practicable for medium hands.
- part 05, "on the trill" (very long): trill fingerings, including **double-fingered trills and trills combined with melody notes** ("when melody notes are difficult to stretch, strike them alone"). Many fingering stacks are transcribed as markups.
- **Moderate**, for ornament fingering.

**Schenker op. 109/110/111 commentaries** (study library): occasional fingering and pedal remarks. **Marginal.**

### 7.3 Other study sources
- **Walker, *Chopin: Life and Times*, ch. 10 "Chopin at the Keyboard"** — study library, part 13, "chapter 10 chopin keyboard". **Relevant.**
  - Thumb freely on black keys, including two adjacent keys with one thumb.
  - Longer fingers passing over shorter ones for legato.
  - Substitutions (Prelude in G: 1 and 4 replaced by 2 and 5).
  - Scales taught starting from B major; black keys as pivots.
  - Finger strength ranking: thumb strongest, then 5, 3, 2, and 4 weakest.
  - 3 and 4 as "Siamese twins".
  - "Flutter pedalling" (up to 7 changes per bar); the pedal as colour, not a cover for legato.
- **Magrath, *The Pianist's Guide to Standard Teaching and Performance Literature*** — study library. Repertoire annotations mentioning fingering and pedal demands by level. **Marginal**, but could support difficulty grading.
- **Shockley, *Contemporary piano*** — study library, part 04, "chapter 01" and chapters 2–4, 9. Pedal mechanics: damper, una corda (grand versus upright), sostenuto, half and partial pedal, extended techniques. **Moderate** for modelling pedal semantics.
- **Jamini, *Harmony & Composition*, lesson 2 "keyboard basics"** — study library, part 03, "lesson 02 keyboard basics". C-major scale fingering: thumb under, 3 over thumb; fingers numbered from the thumb in both hands. **Marginal**, but gives a basic standard-scale check.
- **Mehegan, *Jazz Improvisation 1*, §5 arpeggios** — study library, part 08, "section 5 arpeggios". Arpeggio fingering for 60 scale-tone chords: thumb on the first white note. **Marginal.**
- **Rimm, *The Composer-Pianists*** — study library, part 04, "feinberg scriabin". Remarks on Godowsky and Busoni technique. **Marginal.**
- **Other marginal sources** (mention pedal or keyboard in an orchestration or theory context only):
  - Taylor, *AB Guide* part 2, ch. 21 (study library, part 09, "chapter 21 percussion keyboard")
  - Black & Gerou orchestration, keyboards (study library, part 10, "keyboards")
  - Logier (study library, part 15, "modern instrumentation")
  - Berry, *Form in Music*, fugue (study library, part 13, "ch12 fugue and related genres"), where "pedal" means pedal point, not the piano pedal
  - Lewin's Debussy chapter

---

## 8. Bibliography to acquire next
Ordered by value for the design. "Where cited" points to the chunk with the full reference.

1. **Nakamura, Ono, Sagayama 2014, "Merged-output HMM for piano fingering of both hands"**, ISMIR 2014, pp. 531–536. Joint **hand separation and fingering** from mixed MIDI; chords as fast arpeggios. *The* reference for hand assignment. Cited in Nakamura, Saito and Yoshii 2020, chunks/09-appendices-and-references.md [23] and Randolph 2023, part 13, "cited literature" #91.
2. **Hart, Bosch, Tsai 2000, "Finding optimal piano fingerings"**, *UMAP Journal* 21(2):167–177. A simple, **user-tunable DP with 4 key-colour × finger-pair × interval difficulty tables (672 values)**. An ideal deterministic baseline; the actual table values are needed. Randolph #59; Nakamura [11].
3. **Jacobs 2001, "Refinements to the ergonomic model for keyboard fingering of Parncutt et al."**, *Music Perception* 18(4):505–511. **Physical-distance (mm) spans** and modified rules. Randolph #65; Nakamura [13].
4. **Balliauw, Herremans, Palhazi Cuervo, Sörensen 2017, "A variable neighborhood search algorithm to generate piano fingerings for polyphonic sheet music"**, *Int. Trans. Oper. Res.* 24(3):509–535. Also the 2015 MCM tabu-search paper and the 2014 Antwerp MSc thesis. The **polyphonic extension of Parncutt's rules**, two hands, MusicXML input, public code. Randolph #9–11; Nakamura [3].
5. **Al Kasimi, Nichols, Raphael 2007, "A simple algorithm for automatic generation of polyphonic piano fingerings"**, ISMIR 2007, pp. 355–356. The later version of the AFS model. Randolph #3; Nakamura [1].
6. **Yonebayashi, Kameoka, Sagayama 2007, "Automatic decision of piano fingering based on hidden Markov models"**, IJCAI 2007, pp. 2915–2921. The original fingering HMM, with hand-designed output probabilities that may be useful as untrained priors. Randolph #146; Nakamura [41].
7. **Guan, Zhao, Li 2022, "Estimation of playable piano fingering by pitch-difference fingering match model"**, *EURASIP J. Audio Speech Music Process.* 2022:1–13; and **Guan, Yan, Hsu 2021, "Automatic piano fingerings estimation using recurrent neural networks"**, Nordic SMC 2021, pp. 133–138. BLSTM with **post-hoc ergonomic rule correction** and the "irrational fingering rate" metric. Randolph #56, #57; Srivatsan [5].
   - **Note:** the "pianoplayer" toolkit named in the task brief does **not** appear in any chunk. The GitHub project usually meant is M. Musy's `pianoplayer`, unrelated to Guan. Look it up separately if wanted.
8. **Parncutt, Sloboda, Clarke 1999, "Interdependence of right and left hands in sight-read, written, and rehearsed fingerings of parallel melodic piano music"**, *Aust. J. Psychol.* 51(3):204–210. **Two-hand coupling.** Nakamura [27].
9. **Sloboda, Clarke, Parncutt, Raekallio 1998, "Determinants of finger choice in piano sight-reading"**, *J. Exp. Psychol.: HPP* 24(1):185–203. Empirical transition statistics and the rule factor analysis. Nakamura [36]; Srivatsan [2].
10. **Clarke, Parncutt, Raekallio, Sloboda 1997, "Talking fingers: an interview study of pianists' views on fingering"**, *Musicae Scientiae* 1. Cognitive and interpretive determinants. Srivatsan [7].
11. **Sébastien, Ralambondrainy, Sébastien, Conruyt 2012, "Score analyzer: automatically determining scores difficulty level for instrumental e-learning"**, ISMIR 2012, pp. 571–576. **Difficulty features**, including fingering, hand displacement and polyphony. Nakamura [34].
12. **Nakamura & Yoshii 2018, "Statistical piano reduction controlling performance difficulty"**, *APSIPA Trans.* 7(e13). Uses the fingering model as a **playability prior**. Nakamura [24].
13. **Moryossef, Elazar, Goldberg 2019, "At your fingertips: Automatic piano fingering detection"**. Video-derived noisy fingering data and the alternate PIG splits. Srivatsan [4].
14. **Randolph, Di Eugenio, Badgerow 2021 (SMC, ERR)** and **2023 (SMC, CRF)**; **Pydactyl** (ISMIR LBD 2018); **abcDE / Dactylize** (ISMIR LBD 2016). The originals of Randolph ch. 3–7. Code: `github.com/dvdrndlph/pydactyl`; data: `github.com/dvdrndlph/didactyl`.
15. **Sayegh 1989, "Fingering for string instruments with the optimum path paradigm"**, *Computer Music Journal* 13(3):76–84. The original Viterbi fingering formulation. Randolph #120.
16. **Radicioni & Lombardo 2007, "A constraint-based approach for annotating music scores with gestural information"**, *Constraints* 12(4); and Radicioni's 2005 Turin dissertation. **Phrase-segmentation-based fingering**; chord and mixed nodes. Randolph #102–103.
17. **Lin & Liu 2006, "An intelligent virtual piano tutor"**, ACM VRCIA. A fast Parncutt implementation. Randolph #77.
18. **De Prisco, Zaccagnino, Zaccagnino 2012**, differential evolution with ANFIS. A generalised fingering model. Randolph #42.
19. **Viana, Cavalcanti, Alsina 1998, "Intelligent system for piano fingering learning aid"**. Rules plus a genetic algorithm. Randolph #134.
20. **Wagner 1988** (active and passive spans of more than 200 pianists) and **Matzdorff 1968**. **Measured hand spans** for calibrating MaxPoss and MaxPrac. Cited in Parncutt Parncutt et al. 1997, part 10, "references".
21. **Beringer & Dunhill, *Manual of Scales, Arpeggios, and Broken Chords*** (standard fingerings, already digitised in Didactyl). **Musafia 1971, *The Art of Fingering in Piano Playing***, a modern fingering pedagogy that Nakamura contrasts with Couperin.
22. **Nakamura et al. 2014, "Outer-product HMM and polyphonic MIDI score following"**, *JNMR* 43(2). The source of the **30 ms chord-onset threshold**. Nakamura [22]; Randolph #90.

---
