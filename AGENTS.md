## 0. Bootstrap the complete project documentation

The repository keeps a complete, structured project source package at `downloads/Jumprism-Project-Source.tar.xz`. After cloning, check for `docs/00_PROJECT_CHARTER.md`. If it is absent, extract the source archive into the repository root before planning or implementing work:

```powershell
tar -xJf .\downloads\Jumprism-Project-Source.tar.xz -C .
```

Verify that the `docs/` directory, `UnityProject/` and `tools/` folders are present. The archive contains the full working documentation tree. Do not start implementation if extraction fails or the required source documents are missing.

# Jumprism — Coding-Agent Instructions

These instructions apply to every OpenCode or equivalent coding-agent session in this repository.

## 1. Source of truth

Before planning or editing, read:

1. `README.md`
2. `STATUS.md`
3. `docs/00_PROJECT_CHARTER.md`
4. `docs/03_TECH_STACK_AND_ARCHITECTURE.md`
5. `docs/06_LEVEL_FORMAT_V1.md`
6. `docs/07_DEVELOPMENT_ROADMAP.md`
7. `docs/08_QUALITY_AND_TEST_PLAN.md`
8. The current milestone task in `docs/`.

When instructions conflict, use this priority: explicit user request > this file > project decision documents > milestone task > existing implementation. Report conflicts before making irreversible changes.

## 2. Project constraints

- Target: Windows PC first; Steam distribution is the commercial target.
- Engine decision: Unity 6.3 LTS, subject to a verified installation/OS compatibility gate.
- Main implementation language: C#.
- IDE/debugger: Visual Studio or VS Code; Unity Editor is part of the development toolchain.
- AI is optional authoring assistance. Normal gameplay and published levels must not require a running AI model or online inference service.
- Manual creator tools must remain usable when AI is unavailable, disabled, slow or wrong.
- Do not silently change the engine, render pipeline, input system, physics settings, package versions or level schema.

## 3. Work discipline

1. Inspect the actual repository and Unity project before changing files. Never assume a scene, package, prefab, script, GUID or asset exists.
2. State the exact milestone and acceptance criteria being implemented.
3. Make a short, ordered plan before non-trivial edits.
4. Implement only the active milestone. Do not prematurely add SAM 2, video tracking, online services, Steam Workshop or procedural generation.
5. Keep gameplay, editor UI, geometry, media import, persistence and optional AI boundaries separate.
6. Prefer small cohesive C# types, explicit ownership, predictable lifecycle and testable pure functions.
7. Avoid `FindObjectOfType`, `FindAnyObjectByType`, repeated scene scans, LINQ allocations or new heap objects in hot `Update`/`FixedUpdate` paths unless measured and justified.
8. Use fixed-step physics for movement and moving platforms. Do not let render-frame timing control physics.
9. Treat imported media, JSON, masks, model output and downloaded levels as untrusted input. Validate dimensions, byte sizes, numeric finiteness, paths, schemas and asset references.
10. Do not hide failures. Return actionable errors and preserve the editor state when an operation is cancelled or fails.
11. Keep visible media, segmentation masks, support surfaces and physics colliders as distinct data.
12. Do not auto-create collision for every detected object. A user must approve support surfaces.

## 4. Reliability requirements

- Validate null references, Unity object lifetime and array/list bounds.
- Check `instance == null`/Unity fake-null where applicable before using engine objects.
- Clamp configurable values to documented safe ranges.
- Use stable IDs for serialized objects; do not serialize runtime instance IDs as persistent identities.
- Use atomic or staged writes for level saves; never overwrite the last valid save before a new save is validated.
- Reject malformed or unsupported level schema versions with a clear message.
- Dispose streams, textures, render textures, video players, processes and temporary files on success, cancellation and failure.
- Provide cancellation and timeouts for media import and AI worker calls.
- Keep logs concise but diagnostic: operation, asset ID/path (sanitized), duration, failure category and recovery advice.

## 5. Testing and evidence

- Add EditMode tests for pure geometry, schema validation and serialization.
- Add PlayMode tests for movement, collisions, moving platforms and editor interactions.
- Run the relevant tests after each significant change; run all tests before reporting milestone completion.
- Build the Windows player in CI or locally where the required Unity installation/licence permits.
- Report exact test totals, skipped tests, build status and any manual checks still required.
- Never claim a feature works because a placeholder UI exists.
- If Unity cannot be invoked in the current environment, say so and provide reproducible editor/CLI steps; do not fabricate test results.

## 6. Performance boundaries

- AI segmentation runs during authoring, never inside the gameplay frame loop.
- Cache image dimensions, imported resources, object lookups and geometry where possible.
- Rebuild only dirty masks/geometry, not the entire level after every pointer movement.
- Limit collider vertex counts and simplify polygons/edges before runtime.
- Profile before adding caches or complicated optimisation.
- Place explicit limits on image/video dimensions, file sizes, frame counts and model output.

## 7. Git and documentation

- Keep commits logically scoped with descriptive messages.
- Do not commit private footage, test captures, API keys, local model weights, Unity Library/Temp folders or user media.
- Update the relevant docs when a decision, schema or acceptance criterion changes.
- Record unresolved questions in `STATUS.md` or an issue; do not silently choose a product direction.
- Do not add a licence file until the owner explicitly chooses the repository's source-code licensing strategy.

## 8. Definition of done

A task is complete only when its acceptance criteria pass, relevant automated tests run, resources/lifecycles are checked, error paths are handled, documentation is updated and the result is reported with evidence. Stop at the milestone boundary.