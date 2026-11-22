# 6. RESULTS

## 6.1 Input Representation Ablation

<figure-region kind="render" i="4" />

**Table 1 — restored from the print's text layer** (census site p5
185,95,910,205, classed table high). The values below are verbatim
text-layer cells. Bold: derived deterministically from the caption's
own rule ("values in bold are the closest to the gold labels' score
for that metric") — per column, each candidate cell compared with its
own gold cell: Note ⊕ Octave is closest on Thumb Cross (|331−257| = 74),
Crossed Chord (|28−36| = 8) and Step Spread (|2.78−2.69| = 0.09); the
Lattice row is closest on the remaining six columns. Units: counts
except spreads (semitones per finger) and 4-gram / M_gen (accuracies).

| Model | Thumb Cross (#) | Thumbless Cross (#) | Crossed Chord (#) | Hop (#) | Smear (#) | Step Spread (s/f) | Chord Spread (s/f) | 4-gram (Acc) | M_gen (Acc) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Gold | 331 | 58 | 28 | 155 | 7 | 2.78 | 2.64 | 100 | 100 |
| Bi-LSTM (MIDI) [3]† | 228 | 84 | 47 | 485 | 148 | 2.63 | 2.56 | 66.7 | 27.8 |
| Bi-LSTM (Note ⊕ Octave) | **257** | 90 | **36** | 394 | 135 | **2.69** | 2.61 | 67.0 | 28.3 |
| Bi-LSTM (Note ⊕ Rel Dist) | 218 | 87 | 52 | 390 | 101 | 2.65 | 2.59 | 69.1 | 32.2 |
| Bi-LSTM (Lattice) | 217 | **43** | 40 | **334** | **74** | 2.66 | **2.62** | **69.6** | **33.1** |

[the print's own bold could not be extracted (no bold survives the
text layer); the marks above follow the caption's stated rule, so they
reconstruct the print's intent — flagged here as a derivation, not a
print read]

Table 1. Results on PIG test set for Bi-LSTM model with different input representation schemes. The first row represents the value of each metric under the gold fingerings produced by human annotators. Values in bold are the closest to the gold labels' score for that metric. †This model is based specifically on the neural approach used in [3], but is our own implementation of it. It was similarly reimplemented by [4] in their work. Units for most metrics are simple frequency counts, except for spreads which are in semitones per finger, and 4-gram and M_gen which are accuracies.

We start by investigating the effects of input representations on a simple Bi-LSTM baseline model that outputs independent prediction probabilities at each timestep. Table 1 shows our results across all metrics. Overall the lattice representation does best, followed by note ⊕ octave. This matches our hypothesis that learning a separate embedding for each note on the keyboard leads to overparameterization, and that modular representations are more effective. We also see that relative distance based embeddings lead to fewer hops and smears specifically, which makes sense given that it allows the model to more easily observe when notes are or are not repeated, and therefore whether repeating the same finger would make sense.

## 6.2 Checklist Models

We see in Table 2 that the checklist models tend to do best overall, with Binary Checklist using REINFORCE getting the highest 4-gram score among LSTMs. All the autoregressive systems significantly outperform the Bi-LSTM baselines, but with different tradeoffs in terms of the output patterns we measure. REINFORCE minimizes the number of hops and smears compared to just cross entropy, and in doing so can sometimes boost other metrics as well.

<page n="6" i="5" />

<figure-region kind="rotation" i="5" rotation="cw" />

```
I
II III IV V
```

[Figure 4's rotated axis labels extract here; resolved with the matrix below in 6.3]

<figure-region kind="vector" i="5" />

**Table 2 — restored from the print's text layer** (census site p6
210,100,830,335, classed table high). The extraction's band split
merged the header cells "Smear / Step Spread / Chord Spread" and
"M_gen / 4-gram" into single columns; the layout layer's column
x-positions restore the 10-column shape. ‡ = the print's own footnote
marker on the HMM row.

| Model | Thumb Cross (#) | Thumbless Cross (#) | Crossed Chord (#) | Hop (#) | Smear (#) | Step Spread (s/f) | Chord Spread (s/f) | M_gen (Acc) | 4-gram (Acc) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Gold | 331 | 58 | 28 | 155 | 7 | 2.78 | 2.64 | 100 | 100 |
| Bi-LSTM (MIDI) [3]† | 228 | 84 | 47 | 485 | 148 | 2.63 | 2.56 | 66.7 | 27.8 |
| HMM-3 [3]‡ | 220 | 32 | 31 | 196 | 84 | 2.84 | 2.72 | 70.2 | 39.5 |
| Autoregressive Tagger | 278 | 88 | 49 | 168 | 58 | 2.74 | 2.67 | 68.3 | 36.7 |
| (+REINFORCE) | 274 | 66 | 49 | 59 | 18 | 2.70 | 2.63 | 68.7 | 36.5 |
| Prev Finger Embedding | 271 | 56 | 40 | 127 | 30 | 2.78 | 2.67 | 68.1 | 35.5 |
| (+REINFORCE) | 283 | 53 | 54 | 53 | 12 | 2.81 | 2.76 | 68.4 | 36.5 |
| Binary Checklist | 227 | 72 | 26 | 332 | 74 | 2.67 | 2.58 | 69.3 | 35.8 |
| (+REINFORCE) | 261 | 49 | 26 | 217 | 42 | 2.69 | 2.64 | 69.5 | 37.1 |
| Distance Checklist | 274 | 72 | 37 | 195 | 37 | 2.80 | 2.69 | 68.8 | 36.6 |
| (+REINFORCE) | 284 | 78 | 41 | 134 | 32 | 2.75 | 2.68 | 69.0 | 36.1 |

Table 2. Results on PIG test set for various autoregressive setups, shown with and without using REINFORCE to minimize hops and smears (see Section 4.2). ‡ A third order HMM implementation by [3] is also included – while it ranks similarly as in prior work against LSTMs by M_gen, it falls behind on other metrics.

Note that M_gen fails to fully reflect what are at times substantial differences in model output made apparent from the other metrics. For instance, the Prev Finger Embedding produces fewer hops, smears, and crossed chords than the Autoregressive tagger, likely indicating that it would be far easier for a human to perform despite having worse M_gen. This is especially apparent when comparing the checklist models to the Bi-LSTM baselines. Also while the HMM does strongly on M_gen and 4-gram, we only see its deficits when we look at other metrics which reveal subtle issues such as an especially high finger spread and number of smears, and low utilization of thumb crosses.

## 6.3 Label Confusion

**Label-confusion matrix — restored from the print's text layer**
(census site p6 355,680,260,195, classed table medium; the print gives
this figure no numbered caption — the surrounding prose serves as it;
its rotated I II III IV V axis labels extract as the i=5 rotation
residue above). Arithmetic closure: every column sums to ≈100 (100.1 /
100.0 / 100.1 / 100.1 / 99.9 — column-normalized percentages,
consistent with one-decimal rounding); the rows sum to 117.3 / 114.4 /
86.1 / 67.8 / 114.6 and are not normalized.

| | Predicted Finger I | Predicted Finger II | Predicted Finger III | Predicted Finger IV | Predicted Finger V |
| --- | --- | --- | --- | --- | --- |
| Actual Finger I | 89.3 | 19.1 | 6.6 | 1.8 | 0.5 |
| Actual Finger II | 7.7 | 65.1 | 31.6 | 8.8 | 1.2 |
| Actual Finger III | 1.5 | 13.0 | 43.2 | 25.1 | 3.3 |
| Actual Finger IV | 1.2 | 1.8 | 14.0 | 38.7 | 12.1 |
| Actual Finger V | 0.4 | 1.0 | 4.7 | 25.7 | 82.8 |

This figure shows the confusion matrix for predictions by the Binary Checklist model with REINFORCE. The distribution of misclassifications is fairly different for each finger; adjacent fingers are more likely to be confused than ones that are further apart, reflecting a degree of interchangeability in how pianists use them. We also see that thumb and pinky predictions are most accurate, perhaps because being at the ends of the hand makes them less versatile.

## 6.4 Human Evaluation and Conclusion

**Table 3 — restored from the print's text layer** (census site p6
115,1140,485,110, classed table high).

| Model | Physical Comfort | Mechanical Efficiency | Ease of Learning |
| --- | --- | --- | --- |
| Bi-LSTM (MIDI) [3] | 1.6 | 1.4 | 1.6 |
| Binary Checklist (+REINFORCE) | 2.8 | 2.5 | 2.9 |

Table 3. Human evaluation ratings across various criteria.

Since the ultimate success criteria of our task is to produce fingerings that are well-suited to human performance, we also present results from a small scale round of human evaluation. Specifically, we recruited a college professor of piano with a doctorate in piano performance (who was not involved with the creation of the original dataset) to assess the outputs of two models—the Bi-LSTM (MIDI) [3] and Binary Checklist with REINFORCE—according to three main criteria: physical comfort, mechanical efficiency, and ease of learnability. The pianist was asked to score the models' outputs with respect to each of these categories on a scale of 1-3 for all 10 songs in the validation set (composed by Brahms, Mendelssohn, and Rachmaninoff).

We show the models' average ratings per category in Table 3. The strong contrast suggests that the checklist model is not just predicting gold label fingers more frequently (shown by M_gen) but also that it is producing more locally coherent outputs in a way that substantially improves performability. Furthermore, the correlation of human judgements with our proposed metrics indicates that they are meaningful measurements of output quality.

The pianist also provided qualitative observations which we summarize. He noted that both systems performed best on simpler, repetitive passages such as those found in Mendelssohn. Our model also reportedly produced fewer difficult stretches, which we expect as it is directly aware of interval distances in its input representation. Another observed issue was that the Bi-LSTM's fingerings did not consider tempo, often containing sections that would be impossible to play fast enough, especially for Rachmaninoff pieces. We suspect our model's advantage is from the checklist's dependence on absolute timing, not just the ordering of notes. The pianist also reported that our model more frequently used the same fingering for repeated notes and chords, whereas the Bi-LSTM would often change fingerings without reason, which can also likely be explained by our conditioning on recent predictions instead of decoding them independently. Quantitatively, we found that REINFORCE was able to reduce the number of hops and smears, which was confirmed by the pianist to have a significant impact on playability, emphasizing the importance of integrating these into the training loop instead of purely relying on a traditional likelihood loss.

Finally, he noted that the models particularly struggled on successive chords (as opposed to stepwise passages). Chord transitions require fingerings that are not just comfortable in isolation but also connect together. This requires the model to reason about voices within the same hand, something which may require moving beyond a framework that treats songs as linear sequences of individual notes, and is more sensitive to polyphony.
