<page n="1" i="0" />

<figure-region kind="rotation" i="0" rotation="ccw" />

```
arXiv:2209.05622v1 [cs.LG] 12 Sep 2022
```

<figure-region kind="music" i="0" />

```
ସ
ଵ
ଶ
ଷ
Predictions
```

<figure-region kind="music" i="0" />

```
ଷ
ଶ
ଵ
```

<figure-region kind="vector" i="0" />

**Print transcription, Figure 1** (checklist model architecture; census
site p1 640,385,435,320, classed pitch-bearing medium — the class is a
mis-class of a diagram, no staves exist here). Structured description
from fig.describe --class diagram (run
ops-fig-describe-20261111T004215Z-5ed44dbd, described medium-high;
cross-checked against the text layer):

- Bottom-up flow. The "Song in MIDI" staff — treble clef, one flat, a
  rest, beamed notes, and a chord (decorative illustration, not a
  transcribable music example) — supplies the "Linearized Input Seq"
  x₁-x₆, which feed the "Bi-LSTM" chain: six cells with bidirectional
  arrows between adjacent cells (the two-directional context).
- The Bi-LSTM outputs branch two ways: to the "Past Finger Predictions"
  y₁ y₂ y₃, which point up into the "Checklist" piano-keyboard graphic;
  and through the "MLP" box to y₄, captioned "Current Prediction".
- The checklist keyboard and the prediction path converge via an
  orange rectangle up to a hand icon (the performing hand the checklist
  state describes).
- Text-layer labels read verbatim: Checklist, Past Finger Predictions,
  MLP, Current Prediction, Bi-LSTM, Linearized Input Seq, Song in
  MIDI. The checklist keyboard's key labels read "I II III" in the
  text layer, with the remaining labels lost to a font mis-mapping:
  the fenced Odia-letter residue above is how the print's subscript
  numerals extract (no control characters involved); the certified
  fine-grained readings are the describe run's x₁-x₆ and y₁-y₄.

Figure 1. Visualization of the checklist model. Notes are embedded and then passed to a Bi-LSTM which outputs contextualized embeddings at each timestep. A checklist encodes where the fingers that have recently been used are located based on prior predictions. These are both fed into an MLP which predicts the next finger. [caption restored verbatim from the print's text layer by the verification pass]

# Checklist Models for Improved Output Fluency in Piano Fingering Prediction

| Nikita Srivatsan | Taylor Berg-Kirkpatrick |
| --- | --- |
| Carnegie Mellon University | UC San Diego |
| nsrivats@cmu.edu | tberg@eng.ucsd.edu |

ABSTRACT

In this work we present a new approach for the task of predicting fingerings for piano music. While prior neural approaches have often treated this as a sequence tagging problem with independent predictions, we put forward a checklist system, trained via reinforcement learning, that maintains a representation of recent predictions in addition to a hidden state, allowing it to learn soft constraints on output structure. We also demonstrate that by modifying input representations — which in prior work using neural models have often taken the form of one-hot encodings over individual keys on the piano — to encode relative position on the keyboard to the prior note instead, we can achieve much better performance. [extraction tear at this clause repaired from the print's own text layer: the polyplex band split dropped "neural models have often taken the form of one-hot encod-"] Additionally, we reassess the use of raw per-note labeling precision as an evaluation metric, noting that it does not adequately measure the fluency, i.e. human playability, of a model's output. To this end, we compare methods across several statistics which track the frequency of adjacent finger predictions that while independently reasonable would be physically challenging to perform in sequence, and implement a reinforcement learning strategy to minimize these as part of our training loss. Finally through human expert evaluation, we demonstrate significant gains in performability directly attributable to improvements with respect to these metrics.

© N. Srivatsan and T. Berg-Kirkpatrick. Licensed under a Creative Commons Attribution 4.0 International License (CC BY 4.0). Attribution: N. Srivatsan and T. Berg-Kirkpatrick, "Checklist Models for Improved Output Fluency in Piano Fingering Prediction", in Proc. of the 23rd Int. Society for Music Information Retrieval Conf., Bengaluru, India, 2022. [resolves the census's page-1 decoration site: the print's licence footer]

# 1. INTRODUCTION

While sheet music is often very specific as to what notes a musician must play, it can also be vague about how to play them. Composers generally write notation that focuses on describing the desired musical output, but leave the specifics of how to achieve this with their instrument up to the performer, as these details are outside the scope of their role in the creative process or in some cases beyond their expertise. The piano is an instrument which requires an extensive amount of decision-making on the performer's part. In order to perform a piece of music, a pianist must either consciously or intuitively select which finger to use to play every note in the song. Notes may overlap in duration or even have simultaneous onsets. Since any key on the piano can potentially be played by any finger, there are an exponential number of ways one could perform any given piece; of these however, the majority would be uncomfortable, difficult to play at full tempo, or even physically impossible [1]. The challenge of deciding the most ergonomic fingering to use can be nontrivial for less experienced pianists, who would not yet have the ability to quickly map common musical patterns to conventional fingering strategies, and might produce more consistent and accurate performances if provided with them [2].

However recent work has shown that the use of machine learning techniques may be able to assist in this regard [3–6]. Automatic piano fingering prediction is the task of inferring finger assignments for each note in a symbolic representation of a piano song, given knowledge of which hand is meant to play each note. Human players generally prefer fingerings that balance physical comfort and efficiency [2], yet these criteria are difficult to quantify and highly subjective [7]. Decisions are simultaneously constrained by the current placement of the musician's fingers, and by how that decision may in turn affect possible options for the notes that follow. And while the spatial location of the notes on the keyboard is arguably the most salient factor, timing is also crucial; certain fingerings might be easy for a set of consecutive notes, but impossible if the same notes are in a chord.

<page n="2" i="1" />

<figure-region kind="music" i="1" />

```
i ∈ R ≥0 and offset time
```

[formula residue: tail of the model-contract sentence "onset time t⁻_i ∈ R≥0 and offset time t⁺_i > t⁻_i", restored in full in chunk 03]

Prior approaches to this task have largely treated it as a sequence tagging problem, where the input at each step is simply the pitch of the note and the output is a softmax over indices corresponding to each finger, and have employed methods analogous to those used for part of speech tagging such as LSTMs and HMMs. However, these techniques are limited in their capacity to model output dependencies, and also disregard the amount of elapsed time between notes which can greatly affect a prediction's performability.

We propose an autoregressive approach, where prior predictions are fed back into the network in order to encourage the model to produce fingerings that are not just accurate on average, but also locally fluent and therefore playable. We do this using a checklist which indicates which fingers have either recently been or are currently in use, and where they are on the keyboard relative to the note at the current timestep. By directly exposing this information, the model can more easily learn to restrict its predictions to fingers that are actually free, and also make decisions more directly influenced by the hand's physical placement. We also experiment with an approach that only feeds back in the most recent prediction, as well as an autoregressive tagger which maintains a neural representation of prior predictions using an encoder network.

Furthermore, the evaluation metrics used in prior work do not always correlate with what a pianist might intuitively perceive as "good" predictions. In fact there are many types of desirable (or undesirable) patterns that may emerge in a model's output, which a simplistic metric such as labeling precision may not actually reflect. We demonstrate that it is possible for two models with nearly identical labeling precision to differ dramatically in the frequency of specific types of output subsequences that are physically challenging to play. Therefore, we evaluate on a battery of metrics that collectively convey a more holistic and interpretable overview of fingering quality.

Since these metrics are not always correlated with the labeling precision that a cross entropy loss optimizes, we also investigate incorporating them explicitly into our loss function at train time. While these scores are non-differentiable, we show that reinforcement learning techniques—which have previously seen success in sequence generation tasks outside of music—can successfully optimize them. This, in conjunction with our checklist approach, ensures that the model not only has direct access to the information it would need to avoid undesirable output patterns, but is also encouraged to do so.

Finally, we conduct human evaluations and qualitative analysis which confirm that our approach improves predicted fingerings in ways consistent with our modeling choices, and also suggest directions for future work.

In summary our contributions are as follows: (1) We provide a comparison of various input representations for LSTM models (2) We put forward checklist based approaches which directly incorporate information from previous decisions (3) We introduce several additional metrics which track fluency of output, and demonstrate how to train directly on them using reinforcement learning.
