# java-content

A **content repo** for the `graphl-ux` learning app (sibling repo). It holds the
**Java** concept — notebooks, per-section narration scripts, and the `manifest.json`
that wires sections to scenes — fetched by the app **at runtime** over raw GitHub.

There is **nothing to build, run, or test** here. Correctness is verified by the
`graphl-ux` app consuming this content. (The one executable is
`scripts/colab_generate_audio.ipynb` — a Colab tool that turns `tts/` scripts into
`audio/` `.wav`s.)

## Layout

```
manifest.json   # wires each module: notebook ref + per-section overlay (scene/spine/role/audio/highlight/focus)
notebooks/      # the teaching .ipynb (prose + code source of truth) — 01..12
tts/            # per-section narration scripts (plain spoken prose)
audio/          # generated .wav narration (per-section) — generated on Colab
scenes/         # reserved/empty — real scenes live in the graphl-ux app (src/scenes)
scripts/        # colab_generate_audio.ipynb — tts/*.tts -> audio/*.wav on a Colab GPU
```

## The contract (shared with apache-spark-content)

1. **The notebook is the single source of truth** for a module's prose and code.
   The `manifest.json` only *wires* — it must never duplicate notebook content.
2. The app splits each notebook at every `## ` heading into **sections** (= pages),
   matched to the manifest overlay by **normalized heading text** (case / backticks /
   whitespace insensitive). A heading edit in a notebook must be mirrored here.
3. A section's diagram **images are stripped** by the app — a **scene** replaces them.
4. **Scenes live in `graphl-ux`** (`src/scenes`), authored in TypeScript — not here.
   The Java concept's two maps are `java-jvm` and `java-anatomy`. Here you only
   reference a scene **by id** in the manifest.

## Narration (per-section TTS)

One `.tts` script **per section**, plain spoken prose — what a teacher would say at a
whiteboard. Naming: `tts/<NN>-<SS>-<slug>.tts` → `audio/<NN>-<SS>-<slug>.wav`, where
`NN` is the module number and `SS` the section order. The stem is shared by the
`.tts`, the `.wav`, and the manifest `audio` field. Drop the notebook's "what we'll
cover" overview and the trailing "What's next" outro — those are reading-only.

See `apache-spark-content/CLAUDE.md` "TTS guidelines" for the full style rules (plain
prose, no markdown/code, spell out symbols and acronyms).

## How content is served

The app fetches this repo at runtime over **raw GitHub**:
`https://raw.githubusercontent.com/schemabotview/java-content/main/…`
A content change is live once pushed to `main` — no app rebuild needed.

## Source of notebooks

Notebooks are copied as-is from the runnable curriculum at `~/Projects/java`.

## Status

- Scaffolded; 12 notebooks copied in.
- **Modules 01–07 (core Java)** are scene-wired in `manifest.json` — each section
  mapped to `java-jvm` (runtime topics) or `java-anatomy` (language topics) with
  `spine`/`highlight`/`focus`/`role`. The "What's next" outro of each is intentionally
  unwired.
- **Modules 01–07 are fully narrated** — 96 per-section `tts/*.tts` scripts, each
  wired to a matching `audio/*.wav` stem in the manifest (the notebook overview +
  "What's next" outro dropped per section).
- The per-section `.wav`s still need a Colab generation pass (`tts/` → `audio/`).
- **Modules 08–12 (Spring)** are notebooks only — not wired, because the two scenes
  are Java-language/JVM maps and don't model Spring; those modules need their own
  scenes first.
- Scenes `java-jvm` + `java-anatomy` are authored and registered in graphl-ux.
- **Not yet pushed to GitHub, and not yet added to the app's concept catalog**
  (`graphl-ux/src/content/catalog.ts`) — both are follow-ups once this is reviewed.
