# 4. LEARNING AND EVALUATION

Next we will detail the additional metrics that we introduce, and show how by using reinforcement learning we can train our model to optimize some of them directly.

<page n="4" i="3" />

<figure-region kind="music" i="3" />

```
(r̄ − r(x, ỹ)) X
log p(ỹ i |ỹ <i , x; θ)
L =
```

[formula residue: the REINFORCE loss, restored below in 4.2 as L = (r̄ − r(x, ỹ)) Σᵢ₌₁ᴺ log p(ỹ_i|ỹ_<i, x; θ)]

<figure-region kind="vector" i="3" />

**Print transcription, Figure 3** (fingering patterns tracked by the
fluency metrics; census site p4 135,85,455,195, classed pitch-bearing
medium). Ops-first record: fig.transcribe resolution=fine ran FIRST on
this site (run ops-fig-transcribe-20261111T004152Z-b0b432f7, crop ×2
scaled for the 300-dpi render) and honestly answered
`unreadable-placeholder` with high confidence — the site carries no
staves, noteheads, rhythmic values, dynamics, slurs or articulation
marks, so no LilyPond fragment can be committed without inventing
content. Structured description from fig.describe --class diagram (run
ops-fig-describe-20261111T004416Z-2d3c33ba, described medium-high),
cross-checked against the text layer for printed glyphs:

- A hand outline at top left, fingertip numerals printed across it
  left to right: I, II, III, IV, V; letters L and R beneath the palm.
- Seven small piano-keyboard icons in three rows — row 1: Thumb Cross |
  Hop; row 2: Thumbless Cross | Crossed Chord; row 3: Step Spread |
  Smear | Chord Spread. In each keyboard two keys are shaded orange
  (the describe run: black-key positions in the top two rows, white-key
  positions in the bottom row) and carry Roman numerals; a small curved
  arrow beneath spans the shaded pair where the print draws one.
- Pattern numeral sequences, text-layer primary (the describe run
  flags its right-hand numerals in Hop and Crossed Chord as
  low-resolution): Thumb Cross III I; Hop III III; Thumbless Cross
  III II; Crossed Chord I III II (three numerals in the text layer;
  the grouping across the panel's shaded keys is the one detail both
  instruments leave uncertain — recorded, not normalized); Step Spread
  I IV; Smear I I; Chord Spread I IV.
- Printed annotations: beneath Step Spread, the ratio "= 6/3" (the
  example stretch as ∆p/∆f = 6/3 per the transcribe run's anchors);
  beneath Chord Spread, "/(IV−I)".

Figure 3. Fingering patterns tracked by our fluency metrics. Arrows and shading indicate sequential notes.

## 4.1 Fluency Metrics

We now describe the metrics we use to measure model performance. While research into pianists' fingering strategies shows that decisions are mostly motivated by minimizing ergonomically unfavorable patterns [2, 7], prior work has primarily evaluated on per-note labeling precision averaged over annotators, referred to as M_gen (albeit with some different strategies for handling disagreements between annotators [3]). However, by simply checking if each prediction agrees with any annotator's label for that note in isolation, we miss whether the model's predictions are coherent with one another. For instance, a model may output two adjacent fingers which each agree with an annotator's label, but not the same one, creating a pattern that is harder to play than if it had solely aligned with one of them, despite these having the same M_gen score. Also, a model may output a sequence that agrees with none of the annotators but is at least playable, and yet receives the same score as a physically impossible sequence. This is especially an issue for songs labeled by fewer annotators.

To address this, we expand the scope of our evaluation to include metrics that also measure how fluent model predictions are within themselves. To this end, we track statistics on the frequency of several types of output pattern that would increase the physical difficulty for a human performer, but may not be reflected in the raw labeling precision score. This allows us to more holistically compare models in ways that are both attentive to sustained agreement with annotators, and also expose the specific ways in which their predictions are more or less playable. Visual examples of these are provided in Figure 3 for clarity.

4-gram: Anchored 4-gram precision is the proportion of subsequences of four consecutive notes that were all predicted correctly with respect to a single annotator. This metric measures output coherence, but is still directly tied to the gold labels. Where M_gen scores plateau as they approach the inter-annotator agreement of 71.4, 4-gram precision has more headroom and better stratifies models.

Thumb/Thumbless Crosses and Crossed Chords: Thumb crosses are where a note lower in pitch than the previous is played with a "higher" finger or vice versa, one of those two fingers being a thumb. These shift the hand up or down the keyboard without fully lifting off of the keys. We also track crosses where neither finger is a thumb, which is much less common due to their difficulty. We refer to cases where two fingers are crossed and the notes overlap in time as crossed chords. These are only performed under very specific circumstances, as most chords are not physically playable with crossed fingers.

Hop: These are cases where one finger is used to play two different consecutive notes. While not intuitively difficult, these do not allow the player rest their hand in a fixed spot, and make distances harder to accurately judge, limiting the maximum tempo of a piece, similar to how hunt-and-peck typing is generally slower.

Smear: In a smear, more than one note within a chord is played by a single finger. These are generally utilized when two adjacent keys are both in the chord, but because of the placement of the other notes it is hard to use two separate fingers. While these are extremely rare in our gold labels, naive baselines tend to produce them quite frequently.

Step/Chord Spread: Step spread tracks how far the player must stretch their fingers apart while playing. Specifically, it is the average number of semitones per finger separating any two adjacent but not overlapping notes. For example, if a song contains an E followed by the B a fifth above it, the step spread would be 7 for a model that predicts playing them with fingers 1 and 2 respectively, but only a 3.5 for a model that predicts 1 and 3. We measure similar cases where the notes do overlap in time as chord spread. A large average chord spread is more challenging since it requires stretching the hand over a larger distance.

## 4.2 Loss Functions

Having defined our model as above, we can train it on the cross entropy loss between the predicted distribution p(y|x; θ) and the gold labels by simply backpropagating to obtain parameter gradients and taking gradient descent updates on our training set. Autoregressive models are teacher forced at train time (i.e. conditioned on gold labels from prior timesteps in lieu of the model's own previous predictions), and we decode with beam search at test time.

However, while training on cross entropy can yield strong performance on labeling precision, we find that this is not consistently correlated with other non-differentiable metrics which we may also wish to optimize with respect to. We therefore also investigate using a supplemental loss function based on the REINFORCE algorithm [12] which measures the frequency of undesirable fingering patterns in predicted outputs. While REINFORCE is traditionally associated with environment navigation, it has seen success in sequence generation tasks as well [13–15].

Under this framework, the loss function is formulated as an expected reward, which we approximate by sampling a sequence ỹ from our output distribution (using ancestral sampling) and weighting its reward by its likelihood:

L = (r̄ − r(x, ỹ)) Σᵢ₌₁ᴺ log p(ỹ_i | ỹ_<i, x; θ)

Where r̄ is the rolling average reward over the past 50 examples, serving as our reward baseline. In order to ensure stronger signal during training, we also calculate the REINFORCE loss over 10 note chunks at a time, rather than over an entire song as this would make the attribution of reward to specific decisions less direct. In our experiments, we set the reward to be a function of the number of hops and smears (see Section 4.1):

<page n="5" i="4" />

r(x, y) = exp( − #{i : x_i ≠ x_{i+1}, y_i = y_{i+1}} − #{i : y_i = y_{i+1}, t⁺_i > t⁺_{i+1}} )

[the print's ≠ glyph extracts as "6=" in the layout text layer; restored as ≠ — hops are x_i ≠ x_{i+1} with y_i = y_{i+1}, smears are y_i = y_{i+1} with overlapping onsets]

Because training solely on the REINFORCE objective can lead the model towards degenerate solutions that trivially minimize undesirable patterns while compromising predictive accuracy, we take an approach similar to [15] and [19] where both loss functions are summed into a mixed training objective. This provides a reasonable trade-off between the predictive signal of cross entropy, and the pressure towards fluent output given by REINFORCE. Mixed objective runs are warm-started on just the cross entropy loss to avoid degenerate solutions.
