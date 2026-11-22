# Context discipline while reading a score

Rules for an agent reading a printed score by eye through image and transcription tools (AG-130 of the edition principles, `editorial` profile):

| Rule | Why |
| --- | --- |
| Query with tools (inspect, state, explain, diff) instead of reading whole run directories or score JSON | Model JSON is large; queries are small |
| One measure packet or one comparison pair per image | Detail stays legible, context stays small |
| Use gridded crops for every pitch decision | Ungridded page renders produce pitch misreadings |
| Render and compare after each system, not each note and not only at the end | Defects found per system are cheap |
| Keep open uncertainties in the task record, not in memory | Survives handoff |
| Parallelise by part, with one bar of overlap at seams | Catches ties and slurs across the seam |

Applying them:

- **Tools over run directories:** query a run by view rather than opening its JSON files.
- **One measure per image:** the measure packet (the crop, the gridded context, and what is already known) is the unit; overlay comparisons are one system (or one measure) at a time, never a whole page of pairs.
- **Gridded crops for pitches:** a gridded measure crop, or a zoom on a ledger-line chord, grace, cross-staff beam or unclear accidental; never decide a pitch from an ungridded page render. Without page geometry, align the geometry first rather than guessing a crop.
- **Render-and-compare per system:** after transcribing a system, overlay it against the source before moving on.
- **Uncertainties in the record:** unreadable spots become an annotation (for a test fixture) or an unresolved record (for an edition), not a note-to-self.
- **Parallel by part, one bar of overlap:** when several parts are transcribed concurrently, seam bars are transcribed by both sides so ties and slurs crossing the seam are caught.
