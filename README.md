# Music

A ctxmod module (`github.com/ctxmod/music/context`) with tool-neutral music knowledge for agents that engrave, edit, transcribe, analyse and teach music.

Profiles (none is on by default; select by concern):

- `engraving`: LilyPond notation recipes.
- `transcription`: reading voices from printed scores, and context discipline for transcription agents.
- `editorial`: evidence-driven edition doctrine, editorial hypothesis method and casebook patterns, and Swedish apparatus conventions.
- `keyboard_analysis`: the piano-fingering, hand-separation and pedalling literature, with the CC BY 4.0 Srivatsan et al. (2022) paper as a stored source.
- `tutoring`: a Socratic music-theory tutor (method, output format, progress tracking, and the study, analyze, improvise, pitfalls and steer commands).

Select a profile in a consuming module's `default_profiles`, for example `github.com/ctxmod/music/context/profiles/editorial`, with this module in its `requires`.

Only openly licensed text is stored here. Other books and papers are cited as bibliography only.
