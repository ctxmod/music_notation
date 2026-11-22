# Voice decomposition

How to count voices per staff before entering a single note: count voices per staff from stems, beams, rests and shared noteheads before entering notes. Entering notes into a wrong voice count is the most expensive mistake to unwind later, because every duration, rest and tie decision inherits it.

## The evidence, in order of reliability

1. **Stems** — opposing stem directions on one staff are the primary signal: stems-up and stems-down noteheads in the same measure are two voices. A single notehead with two stems (one up, one down) is *shared* between the voices: one sounding pitch, not two.
2. **Beams** — a beam group belongs to one voice. Beams that would cross a voice boundary (a beamed group whose stems flip direction mid-group) mean the beaming, not the voice count, is unusual; check the second pass before splitting.
3. **Rests** — rests are per voice. A measure with two note columns and a rest under the upper one has two voices, the upper resting where it is silent. Rests that overlap both columns are usually a single-voice print convention, not a third voice.
4. **Shared noteheads** — when two voices sound the same pitch, the print shares one notehead with two stems (or stacks a second stemless head). Read it as two voices at unison, and mind that ties at unison are the classic seam error (see the seam rule in `rules/transcription_context_discipline.md`).

A voice that crosses staves still belongs to one home staff, which is what kneed beams and hand crossings are annotated against. Engraving helper voices (slur or tie carriers) are not music; do not mistake a helper for a sounding voice.

## Practice

- Decide the count from a gridded crop of the measure, then write each voice in absolute pitch with bar checks.
- If the print is genuinely ambiguous (two voices versus chords in one voice, both faithful), pick the simpler reading for the transcription and record the ambiguity as an annotation rather than resolving it silently. Example annotation: "b2. over g2 fis4 can be read as two voices or as chords in one voice; both are faithful to the print".
- Voice count feeds everything downstream (coverage features, empty-voice normalisation, strict notation comparison): a wrong count surfaces as many small diffs rather than one obvious one, which is exactly why it is decided first.
