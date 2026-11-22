<page n="3" i="2" />

<figure-region kind="music" i="2" />

```
c i (x ≤i , ŷ <i ). The model is trained to optimize the cross
output of both LSTMs f i (x <i , y <i ) ⊕ b i (x ≥i ).
```

[formula residue: the checklist-function definition c_i(x_≤i, ŷ_<i), the cross-entropy training sentence, and the autoregressive-tagger LSTM composition f_i(x_<i, y_<i) ⊕ b_i(x_≥i); all restored in the prose below]

<figure-region kind="vector" i="2" />

**Print transcription, Figure 2** (input-representation examples; census
site p3 120,108,480,112, classed decoration low — a census mis-class of
a content figure). Structured description from fig.describe --class
diagram (run ops-fig-describe-20261111T004329Z-545401f4, described
medium-high; cross-checked against the text layer):

- Four side-by-side panels, each pairing a small stack of labeled value
  boxes with a one-octave piano-keyboard glyph beneath; in each keyboard
  exactly one key is filled orange (the current note). Panels 3-4 also
  carry a purple-dotted key (the previous note) and a dashed blue
  arrow from the purple-dotted key to the orange-filled key.
- Panel "Raw Pitch": a single box printed 55 — the current note's MIDI
  pitch as a one-hot index (the text layer's "Note G" labels name the
  example note G₃, MIDI 55; the keyboard's orange key sits right of
  center among the black-key group).
- Panel "Note ⊕ Octave": boxes printed G (green) and 3 (teal) — named
  note sub-embedding + octave sub-embedding.
- Panel "Note ⊕ Relative Distance": boxes printed G and +4 — named
  note + the pitch change p_i − p_{i−1} as a sub-embedding.
- Panel "Lattice": boxes printed −1 and +2 — the vertical (white↔black
  transition) and horizontal (white-key steps) sub-embeddings.
- No key names or numeric tick values are printed on the keyboards
  themselves (describe-run anchor); the example's exact note identities
  beyond the printed box values (55, G, 3, +4, −1, +2) are not
  asserted.

Figure 2. Examples of the input representations we compare. Note that the labels on the vectors are indices corresponding to sub-embeddings shared across notes.

# 3. MODEL

We now describe the basic layout of the models we compare, and detail the specific variations between them. First, we will formalize the basic model contract and explain the basic architecture components that most of them share.

At a high level, each model takes in a MIDI-derived representation of either the left or right hand of a complete piano song, and outputs a predicted finger for each note. More formally, we are given an input sequence of notes x = {x₁, …, x_N}, where each x_i = (p_i, t⁻_i, t⁺_i) is a tuple consisting of a pitch p_i ∈ {1, …, 88} as well as a corresponding onset time t⁻_i ∈ R≥0 and offset time t⁺_i > t⁻_i. From x we must predict a corresponding sequence y = {y₁, …, y_N} where each y_i ∈ {1, 2, 3, 4, 5} is an index representing the finger used to play x_i from thumb to pinky respectively.

To this end, each predictive model defines a distribution p(y|x; θ), generally parameterized by an LSTM variant. Under our non-checklist simple Bi-LSTM baseline, the individual y_i are conditionally independent given x: p(y|x; θ) = ∏ᵢ₌₁ᴺ p(y_i|x; θ). However for our autoregressive models, the likelihood factorizes according to chain rule: p(y|x; θ) = ∏ᵢ₌₁ᴺ p(y_i|y_<i, x; θ).

As shown in Figure 1, the notes are first embedded, and then passed to a Bi-LSTM which outputs a contextualized representation. This contextualized representation is optionally concatenated with a d dimensional vector representation of the checklist c ∈ R^(N×d) before being passed to an MLP which outputs a softmax distribution over finger indices. We can think of the checklist as a function of the past and current notes as well as the past finger predictions c_i(x_≤i, ŷ_<i). The model is trained to optimize the cross entropy loss between this distribution and the true labels.

## 3.1 Input Representations

We now discuss four different strategies to encode the song into an input sequence of vector embeddings (illustrated in Figure 2). These each expose different information to the model which can significantly affect performance.

Raw Pitch: This approach simply represents each note by its pitch p_i, assigning a learnable embedding to each value. This was done by most previous neural implementations [3, 4], and requires learning 88 vector embeddings.

Note ⊕ Octave: Given the fact that a piano keyboard's layout is identical across octaves, we might believe that more important than the exact key being played is the named note itself (e.g. knowing that the next note is a G♯ might be more useful in predicting which finger to use than knowing that it is the MIDI pitch 68). To this end we also evaluate the use of an input representation that consists of an embedding corresponding to the named note concatenated with an embedding corresponding to the octave.

Note ⊕ Relative Distance: If finger choice is primarily a function not of the note's pitch but of its location on the keyboard relative to the other notes in the song, then directly exposing that information saves the model from having to infer it. For this representation, we concatenate an embedding for the named note with one corresponding to the change in pitch p_i − p_{i−1} (see Figure 2). Since large distances between notes require physically lifting the hand from the keys, an act which eliminates any sequential constraints, we cap step sizes at 15 semitones. [5] used a similar approach without the named note embedding.

Lattice: We finally evaluate a "lattice" representation based on the one introduced by [3] (note that this was only used in that work for HMMs and not neural models). As shown in Figure 2, we can think of this as a two-dimensional encoding of relative position, where the first dimension corresponds to the number of white keys between notes (i.e. the horizontal distance), and the second dimension indicates if we have moved from a white key to a black key or vice versa (i.e. the vertical distance). These dimensions are each embedded with corresponding vector embeddings, with a horizontal cutoff of 9 steps. This is the default representation where not otherwise specified.

## 3.2 Checklist Formulation

To encourage the model to produce fluent predictions, rather than simply choosing the most likely finger at each timestep independently, we also consider a series of extensions that feed recent predictions into the MLP alongside the contextualized embeddings produced by the LSTM, in the form of a vector embedding c_i(x_≤i, ŷ_<i) ∈ R^d.

Autoregressive Tagger: We start with a baseline autoregressive variant of our simple Bi-LSTM model in which the forward half of the Bi-LSTM acts as an encoder over prior predictions as well as notes. In this setup, we do away with the Bi-LSTM over notes x, and instead replace it with a forward LSTM f(x, ŷ), and a backward LSTM b(x). At each timestep i, the MLP is fed the concatenated output of both LSTMs f_i(x_<i, y_<i) ⊕ b_i(x_≥i).

Prev Finger Embedding: As a precursor to a full checklist, we also experiment with a neural variant of a Maximum Entropy Markov Model (MEMM) [10]. Under this setup, we map the predicted label of the previous timestep ŷ_{i−1} to a corresponding vector embedding, and concatenate this with the contextualized embedding of x_i produced by the Bi-LSTM, before passing the result to the MLP. This approach only takes into account the most recently used finger and ignores timing, but also requires the fewest additional model parameters.

Binary Checklist: This checklist embedding is a 5 dimensional vector, where each dimension corresponds to a finger. The ith dimension is set to 1 if that finger has recently been predicted for a note of a higher pitch than the current, a −1 if it has recently been predicted for a lower note, or a 0 otherwise. We empirically find that "recent" is best defined as within 100 milliseconds.

Distance Checklist: This representation is a concatenation of 5 vector embeddings, one for each finger. These finger embeddings use the lattice input representation strategy, where the first half corresponds to the horizontal distance between the note that that finger was recently predicted for and the current one, and the second half corresponding to the vertical distance. If a finger has not recently been used, its section in the checklist vector will be filled by a learnable "dummy" embedding.
