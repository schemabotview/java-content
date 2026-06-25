# CLAUDE.md — java-content

A **content repo** (not an app) for the `graphl-ux` engine. It holds the **Java**
concept: notebooks, per-section narration, and the `manifest.json` that wires them.
The app fetches this repo's `manifest.json` + notebooks over the network and renders
them. Read `README.md` first; this file is the working orientation.

This repo is a sibling of `apache-spark-content` and follows the **same contract** —
when in doubt, mirror what that repo does (its `CLAUDE.md`/`DESIGN.md`/`MODULES.md`
are the mature reference, especially the **TTS guidelines** and the **fixed semantic
palette**).

## The core contract (do not break)

1. **The notebook is the single source of truth** for prose and code. `manifest.json`
   only *wires* sections — never duplicate notebook content here.
2. The app splits each notebook at every `## ` heading into sections (= pages) and
   matches them to the manifest by **normalized heading** (lowercase, backticks
   stripped, whitespace collapsed — see graphl-ux `content/module.ts`). A heading edit
   in a notebook must be mirrored in the manifest `heading`.
3. Section diagram **images are stripped** by the app — a **scene** replaces them.
4. **Scenes live in `graphl-ux`** (`src/scenes`), authored in TypeScript — not here.
   The `scenes/` dir is reserved/empty. Reference a scene by **id** only.

## The Java scenes (in graphl-ux, not here)

Two dense maps, ported from NodeMap and registered in `graphl-ux/src/scenes/index.ts`:

- **`java-jvm`** — the runtime: source pipeline (`.java` → `javac` → `.class`) →
  class loader sub-system → JVM memory areas → execution engine (interpreter / JIT /
  GC) → CPU. Node ids are `j-*` (e.g. `javac`, `j-class-file`, `j-interpreter`,
  `j-jit-compiler`, `j-gc`, `j-classloader-subsystem`).
- **`java-anatomy`** — the language grammar: four rhythm rows (Model ▸ Initialize ▸
  Transform ▸ Return) following `[Modifier] Type Name = Value`, plus a Memory column.
  Node ids are `ja-*` (e.g. `ja-type`, `ja-primitive`, `ja-type-var`, `ja-control`,
  `ja-loops`, `ja-methods`, `ja-value`, `ja-record`, `ja-sealed`).

Highlighting a container id lights its children, so spotlight the box (`ja-type`) to
light a whole group. Always check this list of ids before adding a `highlight`/`focus`.

## manifest.json shape

Top level: `concept`, `design`, `scenes[]` (id/title/status), `presentations[]` (one
per module). Each presentation: `id`, `title`, `notebook`, `defaultScene`,
`sections[]`. Each section overlay: `heading` (must match a notebook `## ` heading,
normalized), `scene`, `spine` (bool — feed-mode narration), optional `role` (`"hook"`),
optional `audio` (repo-relative `.wav`), optional `highlight` (scene node ids — AMBER,
rest dim), optional `focus` (node id(s) the camera frames).

## Narration

One `.tts` per section, plain spoken prose. Naming `tts/<NN>-<SS>-<slug>.tts` →
`audio/<NN>-<SS>-<slug>.wav`; the stem is shared by the `.tts`, the `.wav`, and the
manifest `audio` field. **Drop the notebook's "what we'll cover" overview and the
trailing "What's next" outro** — those are reading-only, not narrated. Follow
`apache-spark-content/CLAUDE.md` "TTS guidelines" verbatim (no markdown/code, spell
out symbols/acronyms — e.g. JVM → "java virtual machine", `==` → "double equals").

The source curriculum at `~/Projects/java/tts/` has **one `.tts` per notebook** (a
monologue with the coverage-map intro and outro). Per-section files here are split
from that prose, anchored to the notebook's `## ` headings.

## When wiring a module

1. Confirm the notebook is in `notebooks/` and its `## ` headings are final.
2. Add a `presentations[]` entry; list every section (each becomes a page).
3. Set `scene` per section (`java-jvm` for runtime topics, `java-anatomy` for
   language-grammar topics), `spine` for essentials, `role: "hook"` on the first.
4. Author one `tts/<NN>-<SS>-<slug>.tts` per section, set each `audio`, then run
   `scripts/colab_generate_audio.ipynb` to generate + push the `.wav`s.

## Status

- **Modules 01–07 (core Java)** scene-wired in `manifest.json` (scene/spine/highlight/
  focus/role per section). Module 01 also narrated (17 `.tts` + `audio` refs); 02–07
  wired but not yet narrated. `.wav`s pending Colab.
- **Modules 08–12 (Spring)** are notebooks only — unwired; they need dedicated Spring
  scenes (the two existing scenes are Java-language/JVM maps).
- Not yet pushed; not yet in the app catalog (`catalog.ts`).
