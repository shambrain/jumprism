# Jumprism

**Turn any scene into a world you can jump through.**

Jumprism is a proposed Windows/Steam 2D platformer and in-game creation studio. Its defining idea is to turn photographs and, later, video footage into playable platforming environments. Creators import media, select or draw useful surfaces, review AI-generated object masks, assign gameplay behaviours, test the result, and eventually share creations through Steam Workshop.

> **Project stage:** pre-production / prototype planning. No playable Unity project or implemented gameplay code exists yet. The repository now contains the project instructions, research blueprint, current status and a downloadable archive containing the complete structured project documentation.

## What makes Jumprism different

The player can jump onto scenery that is normally decorative: branches, rooftops, rocks, clouds, signs, vehicles and other objects. A creator should be able to build these worlds without manually modelling every platform.

The project separates four concepts:

1. **Visual media** — the photograph or video shown to the player.
2. **Object mask** — which pixels belong to a selected object.
3. **Support surface** — the editable portion where the character may stand.
4. **Physics collider** — the simplified shape used by the runtime physics system.

AI may propose a mask or surface. Deterministic geometry and physics validate the result. Manual drawing remains a first-class workflow, and AI is optional.

## Current technology decision

| Area | Recommendation |
|---|---|
| Engine | Unity 6.3 LTS, subject to the Windows build/editor compatibility gate |
| Main language | C# |
| IDE/debugger | Visual Studio or VS Code; Unity Editor remains required |
| Coding agent | OpenCode or an equivalent repository-aware agent |
| Image/video segmentation prototype | Meta SAM 2.1, optional local worker |
| Image/mask processing | OpenCV and deterministic C# geometry |
| Windows inference experiment | CPU first; evaluate ONNX Runtime DirectML only after compatibility and performance tests |
| Level format | Versioned JSON manifest plus referenced media/mask/track assets |
| Distribution | Windows PC / Steam; Workshop after local import/export and validation are stable |

Godot remains the primary alternative if the project becomes static-image-first or an open-source engine licence becomes the overriding priority. See the full decision and qualifications in the research blueprint.

## Read these first

1. [Bootstrap instructions](README_FIRST.md)
2. [Coding-agent rules](AGENTS.md)
3. [Project status](STATUS.md)
4. [Research and implementation blueprint](docs/RESEARCH/RESEARCH_AND_IMPLEMENTATION_BLUEPRINT.md)

## Complete structured project pack

The complete 23-file project source is available in:

[`downloads/Jumprism-Project-Source.tar.xz`](downloads/Jumprism-Project-Source.tar.xz)

It contains the full project charter, product/game design, competitor research, technology architecture, AI/image pipeline, video tracking and physics design, level-format proposal, development roadmap, quality/test plan, Steam/Workshop plan, OpenCode master directive, Milestone 0 task, naming notes, research sources, Unity project bootstrap notes and SAM 2 worker notes.

After cloning, extract it into the repository root if the `docs/00_PROJECT_CHARTER.md` file is not already present:

```powershell
tar -xJf .\downloads\Jumprism-Project-Source.tar.xz -C .
```

## First implementation gate

Build only a small manual photograph-based vertical slice first:

- one imported photograph as the visual background;
- one controllable player with run, jump, fall, landing and restart;
- a manual line/polygon tool for creating one-way platforms;
- editable geometry and visible collision overlays;
- save/load of a versioned level document;
- automated tests for jump/landing, one-way collision and save/load.

**Do not begin with SAM 2, video tracking, Steam Workshop, networking or generative AI.** First prove that jumping around on an imported photograph is reliable and enjoyable.

## Important project status

This is a pre-production repository, not a finished game or functioning engine. Current research is a starting point, not proof of market demand, trademark availability or runtime compatibility. See [STATUS.md](STATUS.md) and complete the compatibility gate before creating the Unity project.

No source-code licence has been selected. Do not add a licence file or bundle third-party models until their current terms and required notices have been checked.