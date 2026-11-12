# Jumprism — Project Status

**Last updated:** 2026-11-12  
**Stage:** pre-production / prototype planning  
**Repository:** https://github.com/shambrain/jumprism  
**Repository visibility:** public (as currently configured on GitHub)  
**Product name:** Jumprism (preliminary name search completed; trademark clearance has not been performed)

## Confirmed decisions

- Product concept: a 2D platformer combined with an in-game image/video-based level creator.
- Main engine recommendation: Unity 6.3 LTS, pending exact Windows/editor compatibility verification.
- Main programming language: C#.
- Coding workflow: OpenCode edits source and runs available checks; Unity Editor/CLI performs engine-specific import, test and build tasks.
- AI: optional local SAM 2.1 segmentation worker; manual tools and deterministic geometry remain authoritative.
- First playable deliverable: manual photo-background platforming vertical slice, without AI or video tracking.
- Target platform: Windows PC; Steam is the commercial distribution target.
- Repository name: `jumprism`.
- Complete structured documentation package is committed as `downloads/Jumprism-Project-Source.tar.xz`; the research blueprint and main agent instructions are also directly accessible in the repository.

## Not yet done

- No Unity project has been created.
- No C# gameplay code exists yet.
- No SAM 2 model has been installed or benchmarked on the target Windows PC.
- No prototype has been playtested.
- No Steam page, app ID, Workshop integration or commercial validation exists.
- No source-code licence has been selected; no licence file should be added until the owner decides.

## Gates before implementation

1. Verify the exact Windows 10 build and Unity 6.3 LTS editor compatibility.
2. Install/verify Unity Hub, Unity 6.3 LTS editor, Windows Build Support and Visual Studio/VS Code integration.
3. Extract the complete project documentation package if the `docs/` tree is not present.
4. Create a clean Unity 2D project and commit the initial project settings.
5. Complete Milestone 0 acceptance criteria in `docs/07_DEVELOPMENT_ROADMAP.md`.

## Open decisions

- Final art direction and player character identity.
- Final commercial price and whether any free/trial mode is appropriate.
- Whether the public repository should remain public during commercial development.
- Final licensing for original source code and any bundled third-party components.
- Maximum supported source image/video dimensions and durations after profiling.
