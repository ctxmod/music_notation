# LilyPond notation recipes

How common notation problems are written in LilyPond when engraving an edition from a printed source: beaming groups, grace notes, pedalling, stretched text, finger substitution, and tempo marks with alternatives. Each recipe answers a defect actually found when a printed piano piece was transcribed and engraved.

## Beaming: beat-based, with explicit groups

When the source beams differently from LilyPond's default, switch to beat-based beaming and write **every** group explicitly with `[ ]`. A beaming difference is otherwise visible only by eye in a render-and-compare, because a reading of the music does not compare beaming. Agent-authored notation is absolute and explicit.

## Grace notes: four kinds, fingering at normal size

Distinguish the four kinds: `\slashedGrace` (stroke, no slur), `\acciaccatura` (stroke and slur), `\appoggiatura`, and `\grace`. Fingering on grace notes is reset to normal size with a per-note tweak; otherwise it shrinks to an illegible size. (An engraver event dump reads the grace stroke as a grob property, not an event: the `\acciaccatura` stroke via `Flag.stroke-style`.)

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
