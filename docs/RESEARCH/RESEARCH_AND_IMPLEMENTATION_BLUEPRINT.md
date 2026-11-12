# Jumprism: Research & Implementation Blueprint

**Research checked:** 12 November 2026  
**Product name:** Jumprism (preliminary name search completed; trademark clearance not performed)  
**Target:** Windows PC / Steam, single-player first, built with OpenCode-assisted development

## 1. Executive decision

**Verdict: build a vertical-slice prototype. Do not start by building the full creator engine.**

The idea has a meaningful hook: turn photographs and video footage into playable 2D platforming environments. However, the broad idea of turning images and video into explorable game spaces already exists. The strongest direct overlap found is **Picture Grid Engine**, an HTML5 creation engine distributed on itch.io. Its page describes image/video scenes that players walk through, plus exploration, image-restoration modes and events. The reviewed description does not describe the same workflow of extracting top-facing surfaces, generating platform physics, tracking moving platforms and validating jumps.

The product must therefore be positioned around **platforming on detected visual surfaces**, not simply “turn any image into a game.”

### Recommended technology

- **Engine:** Unity 6.3 LTS.
- **Game and editor language:** C#.
- **Coding agent:** OpenCode (or equivalent) edits the C# and Python source, runs tests and invokes the Unity build/test commands.
- **IDE/debugger:** Visual Studio or VS Code; Unity Editor remains the scene, asset and runtime environment.
- **AI prototype:** Meta SAM 2.1 as an optional offline segmentation worker, initially through Python. Do not make the game depend on a live AI service.
- **Geometry:** deterministic C# algorithms for mask cleanup, contour extraction, top-surface candidates and jump-reachability validation.
- **Sharing:** local level packages first; Steam Workshop only after the level format and validation pipeline are stable.

Unity is the recommended choice because video is a first-class requirement and Unity's Video Player/import pipeline supports common formats such as MP4, MOV and WebM subject to target-platform codec support. Godot 4's built-in VideoStreamPlayer supports Ogg Theora; other formats require an extension or a preprocessing pipeline. Godot is still a strong alternative if the product becomes primarily image-based or if a fully permissive open-source engine licence is the overriding constraint.

**Version and OS gate:** Unity 6.3 LTS is the production line to evaluate, not a beta. Its Windows editor requires Windows 10 21H1 / build 19043 or newer. Verify the installed Windows build before committing. Unity Personal is currently free for individuals/small organisations below the USD 200,000 revenue/funding threshold over the prior 12 months; re-check Unity's current terms before release and if the project grows beyond that tier.

## 2. Competitive landscape

| Product | What overlaps | Important difference |
|---|---|---|
| [Picture Grid Engine](https://factorydigital.itch.io/picturegridengine) | Direct image/video-to-explorable-world creation concept; photographs, artwork and videos can be used as stages. | The documented movement is walking/running around the image, with grid exploration, restoration and events. The reviewed page does not document platform-surface extraction and jumping physics. |
| [Viewfinder](https://store.steampowered.com/app/1382070/Viewfinder/) | Brings photographs, paintings and sketches into the game world. | First-person perception/puzzle game, not a player-facing platformer construction system. |
| [Levelhead](https://store.steampowered.com/app/792710/Levelhead_Platformer_Maker/) | Full platformer level editor, user-generated levels, enemies, hazards and sharing. | Levels are assembled from conventional game objects, not extracted from imported video/photo surfaces. |
| [Cursedland](https://store.steampowered.com/app/3447170/) | Platformer with a level editor, custom-drawn pieces, and importing images/music for certain creator features. | Does not describe the proposed automatic extraction of playable surfaces from footage. |
| [Camera Obscura](https://store.steampowered.com/app/341500/) | Platform-related visual manipulation and a custom level editor. | Uses an afterimage mechanic to manipulate the environment; not a general photo/video-to-platform generator. |

**Competitive conclusion:** there is a close conceptual predecessor (Picture Grid Engine), so do not claim that no one has turned images/videos into explorable worlds. The narrower combination of (1) photo/video import, (2) prompted segmentation, (3) top-surface/platform extraction, (4) stable moving-platform physics and (5) jump-validated level generation remains the product's proposed differentiator. This research is not proof that no other prototype, unreleased game or obscure project exists.

## 3. Product definition and gameplay loop

The product is both a platform game and a playable-world creator. The editor should be part of the shipped application; players should not need Unity or programming knowledge.

### Player-facing promise

**“Turn any scene into a world you can jump through.”**

The creator imports an image or supported video, selects or draws surfaces, assigns behaviours, tests the level and publishes a playable result.

### Three authoring modes

1. **Manual mode (must work without AI):** freehand platform line, polygon, brush/eraser mask, spawn point, hazard zone and enemy spawn placement.
2. **AI-assisted mode:** click or box-select an object. Segmentation produces a mask. The editor proposes possible platform surfaces, and the creator approves/edits them.
3. **Video tracking mode (advanced):** select an object on a chosen frame, propagate its mask through a short clip, derive its surface path, review it, then bake a stable moving-platform track.

### Gameplay object types

- Background-only image/video layer.
- One-way platform surface.
- Solid platform / wall.
- Moving platform (initially rigid translation; rotation later).
- Hazard volume or damaging surface.
- Player spawn and checkpoint.
- Enemy spawn marker and trigger zone.
- Goal/exit.

Do not automatically turn every visible object into a collision body. A tree's visible mask is not the same thing as its walkable upper branch. The visible object, segmentation mask, support surface and physics collider must remain separate records.

## 4. How the image-to-platform algorithm should work

### Image pipeline

1. Import and normalize the image. Preserve the source dimensions and record the scale used in the game world.
2. The creator clicks/boxes an object or uses manual drawing.
3. A segmentation model returns a binary object mask.
4. Apply deterministic mask cleanup: remove tiny connected components, close small gaps, remove isolated pixels and clamp the mask to the image bounds.
5. Trace contours and simplify them to a manageable number of points (for example, Douglas-Peucker/Ramer-Douglas-Peucker simplification).
6. Analyze the upper contour and find connected runs with acceptable slope, minimum length, minimum support width and sufficient clearance above the surface.
7. Display candidates as editable overlays. The creator accepts or rejects each one.
8. Convert accepted support runs to low-complexity physics surfaces, normally one-way edge/segment colliders.
9. Save the source reference, mask, surface geometry and object properties in the level document.

### Important physics design

Use a character controller with a predictable jump model and low-complexity platform colliders. For Unity 2D, a typical implementation is Rigidbody2D + CapsuleCollider2D for the player, and EdgeCollider2D/PolygonCollider2D plus PlatformEffector2D or a deliberately controlled one-way collision solution for surfaces.

Do not make each pixel a collider. Do not rebuild thousands of collider vertices every rendered frame. Keep a visible object sprite separate from its support collider. Physics updates must be deterministic and occur in FixedUpdate; visual tracking can be interpolated between sampled frames.

### The algorithm that makes a level playable

Create a **jump-reachability graph**:

- Each accepted platform is a graph node.
- For each candidate takeoff/landing pair, simulate the player's jump under the configured gravity, horizontal speed and jump impulse.
- Add a directed edge only when the trajectory can reach the target platform with adequate landing clearance.
- Check that the goal is reachable from the spawn.
- Suggest a path through reachable surfaces; show unreachable surfaces as warnings rather than quietly making them solid.
- Only suggest enemy positions on reachable platforms with enough headroom and distance from mandatory landings.

This deterministic solver is more trustworthy than asking a language model to invent coordinates. AI can rank or describe candidate paths later, but the physics solver must validate them.

## 5. Video: staged support, not magic auto-platforming

Video backgrounds and moving collision surfaces are different problems.

### First supported footage

For the first video prototype, accept short, fixed-camera footage with no hard cuts, low camera shake and one clearly visible moving object. Play the video as the visual background. Let the creator select the object on one frame, then track it across sampled frames.

### Moving-platform strategy

- Segment/track at authoring time, not during normal gameplay.
- Store a small set of timestamped object poses and platform-surface points in object-local coordinates.
- Smooth noisy tracking and allow manual keyframe corrections.
- During gameplay, interpolate the platform pose and move its physics body at the physics tick.
- Keep the collider's local shape stable for rigid objects; update only the platform transform where possible.
- Detect tracking loss, large jumps in pose, scene cuts and mask fragmentation. Mark the track invalid for manual repair rather than producing a dangerous collider.

Start with rigid or near-rigid translation. Deforming clouds, foliage, water, camera pans and occlusion-heavy footage should be later research features. A moving cloud that changes shape every frame is not a dependable first collision surface.

## 6. Free AI models and hardware realities

### Best candidate found: Meta SAM 2 / SAM 2.1

Meta describes SAM 2 as a promptable model for segmenting objects in images and videos using clicks, boxes or masks, with video tracking and corrections across frames. The official repository states the SAM 2 model checkpoints and code are Apache 2.0 licensed. That is a useful commercial-friendly starting point, subject to maintaining the required notices and checking all other dependencies separately.

Use a tiny/small checkpoint for the first feasibility test. It is an object-segmentation tool, not a platform detector: it can identify a selected branch or cloud, but it does not know whether the object's upper edge is safe or fun to land on. The geometry pipeline and human confirmation solve that.

### Windows + AMD caveat

The target development PC in the project context is Windows 10 with a Radeon RX 6700 XT 12 GB, Ryzen 5 7600X and 32 GB RAM. AMD's current Windows ROCm/HIP documentation does not make the RX 6700 XT a supported HIP SDK target, and its Windows HIP SDK support is tied to Windows 11. Do not plan on CUDA or assume standard PyTorch ROCm will accelerate SAM 2 on this machine.

ONNX Runtime's DirectML execution provider can accelerate ONNX models on DirectX 12-class hardware, including AMD GPUs, and is compatible with Windows 10 1903+. However, Microsoft now describes DirectML as being in sustained engineering and recommends WinML for newer Windows 11 deployments. More importantly, converting SAM 2 to ONNX and ensuring its operators work with DirectML is a separate engineering task; it is not automatic.

Practical plan:

- Keep the model optional and run it only in the creator workflow.
- Prototype SAM 2.1 with CPU fallback first; measure actual import/segmentation times on the target PC.
- In parallel, test an exported ONNX version with ONNX Runtime DirectML.
- If export or DirectML compatibility is unreliable, keep CPU fallback and ship manual tools; do not block the game on AI acceleration.
- Never run the segmentation model every gameplay frame. Bake accepted masks and tracks into the level package.

## 7. Engine choice in more detail

### Unity 6.3 LTS + C# (recommended)

Why:

- Common media import and playback support is stronger for a video-first product.
- C# is a good fit for a typed, modular tool application and works naturally with Visual Studio and OpenCode.
- Unity has a mature 2D physics system, runtime UI options, and an inference package ecosystem.
- Steam distribution and Steam Workshop integration are established workflows.

Trade-offs:

- Proprietary engine and Personal-tier financial threshold.
- Heavier editor and package ecosystem than Godot.
- Native video support is still platform/codec dependent; validate the actual Windows Steam player build, not just Editor playback.

### Godot 4.7.2 (best alternative)

Why:

- MIT-licensed, no engine royalty or revenue tier; include the required notices.
- Excellent 2D foundation, custom drawing and in-engine editor plugins.
- Built-in image-to-collision-polygon workflow and one-way collision support provide useful primitives.
- A 2026 Reddit community example, Map Builder, demonstrates a substantial custom level editor operating directly inside the Godot viewport. Another Godot developer documented a player-facing level editor that stores JSON and converts it into Godot scenes at runtime.

Trade-off: the built-in video player does not natively play ordinary MP4/H.264 or WebM in core Godot 4; use an extension or a controlled preprocessing/import pipeline.

### GameMaker

Keep it in consideration only if the concept is reduced to a conventional image-based platformer with a small built-in editor. Its GML and room workflow are excellent for ordinary 2D games, but this project is unusually editor-, media- and AI-heavy. A commercial GameMaker licence is currently required for paid releases.

## 8. Recommended build sequence

**Milestone 0 — prove the jump is fun**

- Start a Unity 6.3 LTS 2D project.
- Use one photograph as a background.
- Build a placeholder player with run/jump/land/fall/restart.
- Add manual line drawing for one-way platforms.
- Save and reload the drawn platform geometry.
- Prove that jumping and landing feel reliable before any AI work.

**Milestone 1 — image editor**

- Import PNG/JPG.
- Add pan/zoom canvas, selection, polygon tool, platform-line tool, brush, eraser and undo/redo.
- Display physics debug overlays.
- Add start, goal, checkpoint, hazard and enemy-spawn markers.
- Save/load a versioned JSON level document plus image/mask assets.

**Milestone 2 — AI image segmentation**

- Add the SAM 2.1 helper as an optional local process.
- Accept clicks/boxes and return a mask image plus confidence/metadata.
- Add correction clicks and manual mask editing.
- Extract candidate top surfaces and let the creator accept/reject them.
- Add import cancellation, timeout, invalid file handling and memory limits.

**Milestone 3 — jump validator and assisted generation**

- Build the jump-reachability graph.
- Display reachable platform connections and failed jumps.
- Add a route generator that selects a spawn-to-goal path.
- Place hazards/enemies only after route validation.
- Make every generated level playable without manual code changes.

**Milestone 4 — simple video backgrounds**

- Import short MP4 footage using Unity's supported Windows codecs.
- Preview, scrub and loop the footage.
- Sample frames for AI segmentation.
- Begin with video as a moving visual backdrop and static/manual platforms.

**Milestone 5 — moving video platforms**

- Track one prompted rigid object across frames.
- Bake a small set of surface keyframes.
- Provide manual correction UI.
- Drive a kinematic platform at fixed physics ticks.
- Test landing, riding, jumping off, tracking loss and loop boundaries.

**Milestone 6 — Steam product**

- Ship a polished campaign that showcases the mechanic; do not depend on Workshop content to make the game worthwhile.
- Add local save/export/import first.
- Add Steam Workshop after validating and versioning level packages. Steamworks offers ready-to-use Workshop integration for user uploads/downloads, but the game still needs validation and curation safeguards.
- Add discovery, creator profiles, ratings/favourites and moderation only after the creation loop is proven.

## 9. Level package and project structure

Use a versioned, data-driven level format. Do not save Unity scene files as the public interchange format.

Suggested package structure:

```text
MyLevel.fpl/
  level.json
  media/
    background.png
    background_video.mp4
  masks/
    object_0001.png
  geometry/
    platforms.json
    tracks.json
  preview.png
```

Suggested source folders:

```text
Assets/
  Art/
  Audio/
  Prefabs/
    Player/
    Platforms/
    Hazards/
    Enemies/
  Scenes/
    Boot/
    Menus/
    Creator/
    Gameplay/
  Scripts/
    Core/
    Gameplay/
    Physics/
    CreatorUI/
    MediaImport/
    Segmentation/
    Geometry/
    LevelValidation/
    Persistence/
    Steam/
  Tests/
    EditMode/
    PlayMode/
  StreamingAssets/
    Models/
```

The level document should contain a schema version, project GUID, source media dimensions, coordinate scale, object IDs, object type, local-space geometry, platform properties, video timestamps, tracking confidence, and asset hashes. Validate every read: reject unsupported schema versions, missing assets, invalid/non-finite points, empty geometry, oversized dimensions and malformed tracks. Never silently load corrupt user content.

## 10. Release quality and validation gates

Before adding more features, automate these cases:

- Player lands on the platform from above and passes through it from below where one-way behaviour is intended.
- Narrow, sloped and discontinuous surfaces behave consistently.
- Player cannot stand on decorative pixels that were not approved as support surfaces.
- Jump graph agrees with actual physics tests at several jump speeds/heights.
- Start-to-goal route is reachable; unreachable goals are rejected.
- Moving platform carries the player without jitter, teleporting or unexpected detachment.
- Tracking loss and scene cuts stop or invalidate the platform safely.
- Video loop start/end does not teleport the collider unexpectedly.
- Undo/redo restores media selection, mask, geometry and properties.
- Save/load round-trip preserves coordinates and assets exactly.
- Missing media, bad videos, invalid JSON, excessive dimensions and cancelled inference show recoverable errors.
- The final Windows build is tested outside the Unity Editor.

## 11. OpenCode's first assignment

Paste this into the project after creating a blank Unity 6.3 LTS 2D project:

> You are the senior Unity 6.3 LTS C# engineer for this project. Build only Milestone 0 and the foundation required for Milestone 1. Do not implement SAM 2, video tracking, Steam Workshop, procedural level generation or networking yet.
>
> First inspect the complete Unity project, installed packages, target build settings and existing files. Produce a short implementation plan and identify any incompatible project settings before changing files. Then create a clean, documented folder and assembly structure for Core, Gameplay, Physics, CreatorUI, MediaImport, Geometry, Persistence and Tests. Use C# with strict null/input validation, explicit ownership and lifecycle cleanup, no per-frame allocations in gameplay loops, no repeated scene searches in Update/FixedUpdate, and no unbounded file reads.
>
> Implement a minimal 2D player with reliable movement, jump, grounded state, fall/restart and camera framing. Implement a manual platform-line drawing tool over a background image, with selectable/editable points, undo/redo, a clear collision debug overlay and one-way platform behaviour. Save/load the level through a versioned JSON document. Separate visible media from collision geometry. Add EditMode/PlayMode tests for jumping, landing, one-way collision, invalid geometry, save/load round-trips and missing media. Use a fixed-step physics model. Run the project's tests and a Windows development build, report exact pass/fail results and fix all failures before claiming completion. Do not add placeholder features that appear functional but are not implemented.
>
> Keep the code readable and modular. Document the level schema, player movement assumptions, collision coordinate system, error handling and build/test commands. Do not silently change engine version or package versions. Stop after the vertical slice is testable and provide screenshots/instructions for manual playtesting.

## 12. Research sources

### Direct competitors and market examples

- [Picture Grid Engine v1.1 — itch.io](https://factorydigital.itch.io/picturegridengine)
- [Viewfinder — Steam](https://store.steampowered.com/app/1382070/Viewfinder/)
- [Levelhead — Steam](https://store.steampowered.com/app/792710/Levelhead_Platformer_Maker/)
- [Cursedland — Steam](https://store.steampowered.com/app/3447170/)
- [Camera Obscura — Steam](https://store.steampowered.com/app/341500/)

### Engine and technical documentation

- [Unity 6.3 LTS release](https://unity.com/blog/unity-6-3-lts-is-now-available)
- [Unity 6.3 system requirements](https://docs.unity.com/en-us/engine/6000.3/manual/get-started/install-and-upgrade/getting-started-installing-unity/system-requirements)
- [Unity video file compatibility](https://docs.unity.com/en-us/engine/6000.3/manual/video/sources/file-compatibility)
- [Unity Personal terms / eligibility](https://unity.com/products/unity-personal)
- [Godot licence](https://godotengine.org/license/)
- [Godot 4.7.2 stable release archive](https://godotengine.org/download/archive/)
- [Godot video playback formats](https://docs.godotengine.org/en/latest/tutorials/animation/playing_videos.html)
- [Godot collision shapes and image-to-polygon generation](https://docs.godotengine.org/en/latest/tutorials/physics/collision_shapes_2d.html)
- [Godot custom editor plugins](https://docs.godotengine.org/en/stable/tutorials/plugins/editor/making_plugins.html)
- [Steam Workshop documentation](https://partner.steamgames.com/doc/features/workshop)

### AI, inference and video tooling

- [Meta SAM 2 project](https://ai.meta.com/research/sam2/)
- [SAM 2 official repository and checkpoint licence](https://github.com/facebookresearch/sam2)
- [ONNX Runtime DirectML provider](https://onnxruntime.ai/docs/execution-providers/DirectML-ExecutionProvider.html)
- [ONNX Runtime Windows / WinML guidance](https://onnxruntime.ai/docs/get-started/with-windows.html)
- [AMD ROCm Windows system requirements](https://rocm.docs.amd.com/projects/radeon-ryzen/en/docs-7.1.1/docs/shared/hipsdk/reference/system-requirements.html)
- [FFmpeg licensing information](https://www.ffmpeg.org/legal.html)

### Community / real developer discussions

- [Reddit: Godot Map Builder — in-editor level editor](https://www.reddit.com/r/godot/comments/1utnnhh/i_built_a_trenchbroomstyle_level_editor_directly/)
- [Reddit: player-facing editor with JSON levels converted to Godot scenes](https://www.reddit.com/r/godot/comments/1vct0ow/i_made_a_level_editor_for_my_players_json_files/)
- [Reddit: Godot vs Unity for a solo 2D developer](https://www.reddit.com/r/IndieGameDevs/comments/1w3ckt6/unity_or_godot_for_a_solo_indie_developer/)
- [Godot Forum: standalone level editor and whether users need Godot installed](https://forum.godotengine.org/t/before-i-go-down-a-rabbit-hole/128703)
- [Godot Forum: custom main-screen editor plugin](https://forum.godotengine.org/t/making-a-new-tab-like-the-2d-or-3d-tab-using-a-custom-plugin/95612)

---

**Final recommendation:** prototype the manual image-to-platform loop first, then add AI segmentation, then simple video, then tracked moving platforms. If the first prototype is fun and reliable, there is a differentiated product worth developing. If manual image-based jumping is not enjoyable, AI and video will not rescue it.