# LilyPond notation recipes

How common notation problems are written in LilyPond when engraving an edition from a printed source: beaming groups, grace notes, pedalling, stretched text, finger substitution, and tempo marks with alternatives. Each recipe answers a defect actually found when a printed piano piece was transcribed and engraved.

## Version baseline: LilyPond 2.26

The recipes target **LilyPond 2.26** (2.26.0, released 2026-04-21, the
current stable series; 2.27.x is the development series). Every
notation source states its version (`\version "2.26.0"`); files written
for 2.24 are upgraded with `convert-ly -e` and the result reviewed as a
semantic diff (P-09), never accepted blind. The 2.24 → 2.26 changes that
touch these recipes:

- **Beaming properties renamed.** `baseMoment` is deprecated in favour of
  `beatBase` (and `measureLengthAsMoment` → `measureLength`,
  `minimumPageTurnLength` → `pageTurnMinimumRestLength`, among others);
  `timeSignatureSettings` now uses `beatBase`, a change `convert-ly` does
  not make for you. Automatic beam subdivision was reworked
  (`subdivideBeams` plus `beamMinimumSubdivision`,
  `beamMaximumSubdivision`, `respectIncompleteBeams`), and the 2/8
  auto-beaming default changed. Explicit `[ ]` groups (below) are
  immune to all of this, which is one more reason to write them.
- **Slashed beamed graces.** `\slashedGrace` on a beamed group now prints
  a slashed beam (`beam::slashed-stencil`), not only a slashed flag.
- **Wider default margins.** Default page margins grew, following
  publisher practice and Gould; a 2.24 layout reflows (systems, page
  breaks, page turns) under 2.26. Re-review page turns (P-20) after an
  upgrade, or restore the old `\paper` margins deliberately while
  reproducing an earlier engraving.
- **Markup renames.** `\roman` → `\serif`, `\medium` →
  `\normal-weight`, and `\text` is removed; tempo and spanner texts that
  set their font through markup need these spellings.
- **Bar checks create contexts.** A `|` can now implicitly create a
  context; a bar check at the very start of a music expression meant for
  an existing context is worth a second look.

## Beaming: beat-based, with explicit groups

When the source beams differently from LilyPond's default, switch to beat-based beaming and write **every** group explicitly with `[ ]`. A beaming difference is otherwise visible only by eye in a render-and-compare, because a reading of the music does not compare beaming. Agent-authored notation is absolute and explicit.

## Grace notes: four kinds, fingering at normal size

Distinguish the four kinds: `\slashedGrace` (stroke, no slur), `\acciaccatura` (stroke and slur), `\appoggiatura`, and `\grace`. Fingering on grace notes is reset to normal size with a per-note tweak; otherwise it shrinks to an illegible size. (An engraver event dump reads the grace stroke as a grob property, not an event: on an unbeamed grace the `\acciaccatura` stroke via `Flag.stroke-style`; on a beamed `\slashedGrace` group in 2.26 the slash belongs to the beam stencil, so a check that looks only at `Flag` misses it.)

## Pedal: the target's style; re-pedal without a printed release

Pedal follows the target edition's style. A re-pedal without a printed release is written as a one-off sustain string. The mixed style, `pedalSustainStyle = #'mixed`, prints "Ped." with a bracket for its length: the compact form that shows every change. Keep dynamics, pedal and between-staff marks in a dedicated `dynamics` variable rendered in its own `Dynamics` context below the lower staff (so a pedal-only `Dynamics` line can carry just the pedal events), never inline in the note streams. A dump reads `SustainEvent` with its direction and span.

## Stretched *ri-tar-dan-do*: a text spanner, not lyrics

Rhythmically stretched text such as *ri-tar-dan-do* is written as a **TextSpanner**, not as lyrics: a `Lyrics` context with centre affinity collides with a dynamic, and the text must not share the lyrics' mechanics. Write the spanner in the `dynamics` variable, not in the note stream.

## Finger substitution: `\finger \markup \tie "45"`

A 4–5 substitution on one note is written `\finger \markup \tie "45"`. An event dump renders a fingering as its written text (a plain digit as a digit, `\finger` markup as text), so "45" with its tie survives the dump untouched. Ordinary fingerings stay bare `-n` post-events.

## Tempo with alternatives: the movement's tempo text

A movement with tempo changes inside it ("Andante – Allegro – Tempo I") is carried by the movement's tempo text in the edition's metadata, not by marks in the music: it is metadata, not music. The text returns as part of the movement's heading on the engraved page, styled as a tempo mark. For example:

```kdl
part I {
    tempo "Andante – Allegro – Tempo I"
}
```

## The boundary with appearance tweaks

The recipes above write the music's **notation**. Offsetting and shaping what is already written (`\shape`, directions, spacing) is a separate layer of appearance adjustments, kept apart from the notation and leaving the music's meaning unchanged (edition principle P-12, `editorial` profile).

## Sources

- https://lilypond.org/ — release news: 2.26.0 stable (2026-04-21), 2.27.5 development (2026-11-15); accessed 2026-11-23.
- https://lilypond.org/doc/v2.26/Documentation/changes/index.html — "Changes" for 2.26.0 since 2.24: rhythm (beam subdivision, `\slashedGrace` beams, 2/8 auto-beaming), major changes (default margins), text and fonts (`\serif`, `\normal-weight`, `\text` removed), miscellaneous (deprecated `…AsMoment`/`baseMoment` properties, `timeSignatureSettings` not handled by `convert-ly`); accessed 2026-11-23.
