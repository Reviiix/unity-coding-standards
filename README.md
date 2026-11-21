# Unity Coding Standards

These standards apply to every Unity project, for every new feature, fix and refactor. Check each change against them before committing. You don't need to rewrite existing code that breaks a standard unless asked, but any code you add or touch must follow them.

Each project records its target platform (and anything else project-specific) in its own README. If it's missing, ask.

## Architecture

- **One entry point.** A single `GameInitialiser` (`[DefaultExecutionOrder(-1000)]`) controls start-up:
  - `Awake` creates the plain C# services and registers every service.
  - `Start` initialises the systems in dependency order.
  - Systems expose an `Initialise(...)` method rather than doing setup in their own `Awake`/`Start`.
- **Service locator, not singletons.** Services are registered with a `ServiceLocator` (one instance per type) and fetched from it. Don't use static `Instance` fields, and don't search the scene with `Find...` at runtime.
- **Logic in plain C#.** Keep rules, maths and state in plain C# classes or structs (no `MonoBehaviour`), so they can be unit tested. Components are thin: they wire up, draw and forward input.
- **Additive scenes for separate concerns.** A self-contained presentation (such as a 3D view) can live in its own scene. Load it additively from the initialiser, register it, and pass it what it needs. It must be workable when opened on its own.
- **Events for change.** Subscribe in `Initialise`, unsubscribe in `OnDestroy`. Raise events only when a value actually changes.

## Code style

- **Names explain themselves.** Write full words and avoid abbreviations. Name methods for what they do (`TryGetCellIndex`, `ShowSetup`), and name booleans as statements (`IsFitted`, `HasAngles`).
- **One-line `/// <summary>` on every class and method**, private ones included, in plain English. Keep other comments to a minimum: only for something the code can't say (a value that must match a shader, or a non-obvious trap).
- **Private by default.** Use `[SerializeField] private` only for what a designer needs to tweak in the Inspector, and give each one a `[Tooltip]`. Expose state through read-only properties.
- **Namespaces match folders** (`Project.Grid`, `Project.UI.Screens`, and so on), all in one project assembly plus a test assembly.
- **Constants over magic numbers**, each with a summary saying what it means. When a value must match elsewhere (C# and HLSL), say so at both ends.
- **Enums that are saved** (PlayerPrefs or serialized) are stored by number. Add new values at the end, never in the middle.

## Performance (mobile first)

- **Allocate at start-up, never per frame.** Size buffers for the largest case once. A longer start-up is acceptable.
- **Cache references and precompute** anything that doesn't change: tables, labels, meshes and tweens built once and replayed.
- **No per-frame garbage.** Use `StringBuilder` and `TMP_Text.SetText(StringBuilder)` for text. Avoid LINQ, closures and boxing in hot paths.
- **Avoid `Update()`** where a coroutine, event, tween callback or one-off ticker is simpler, but never at the cost of stability or readability. Per-frame work should run only while something actually moves.
- **Pool objects** (and particles) rather than instantiating at runtime.
- **Let the GPU do bulk work.** Use one draw for many identical things (a single quad or instanced mesh reading per-item data from a texture) instead of thousands of objects.
- **Save battery.** Drop the frame rate when nothing is moving (`OnDemandRendering`), and cap render resolution on phones.
- **Assets:** set sensible maximum texture sizes, and use compression that doesn't blur or lose quality. Remove unused assets, and use culling.

## Unity practice

- **Prefabs and prefab variants** for anything reused, built so they're easy to edit later. Edit prefabs and scenes from scripts with `PrefabUtility.LoadPrefabContents` and `SerializedObject`, then save.
- **Keep scene and prefab diffs to the intended change.** Check `git diff` before committing, because saving can write layout-driven values you didn't mean to change.
- **Clear scene hierarchy** with no unused objects. UI under a camera canvas goes on the UI layer.
- **Designer data in ScriptableObjects.** Use a settings asset for limits, timings and starting values, and a theme asset for colours, shapes and animation curves.
- **Player choices** go in `PlayerPrefs` under namespaced keys (`Options.*`). Load each one clamped to the current limits, falling back to its starting value in the settings asset.
- **One look source.** Colours come from theme roles, never hard-coded. Visual-only options (shapes, 3D view) live in the presentation layer and never touch the rules.

## Changing existing code

- **Keep the core gameplay loop and look and feel the same** when refactoring, unless asked otherwise.
- **The project must stay workable by developers unassisted afterwards.** Keep the README (architecture, how things work, common tasks) and any agent handoff notes accurate in the same change.

## Testing

- **Unit test the plain C# logic** with EditMode tests: one fixture per class, test names that read as behaviour, and a summary on each test. Run with `--mode EditMode`.
- **Check behaviour in the Editor in Play mode.** Build to a device only when there is a device-specific reason.
- **Compile player scripts for each target platform** before finishing.
- **When testing in Play mode, restore any saved player data you changed**, and leave the Editor out of Play mode.

## Git workflow (git flow)

- Before editing, check that `git status` is clean, then branch off `develop`: `feature/<short-name>`, or `bugfix/`, `release/` or `hotfix/` as appropriate. Never commit directly to `develop` or `main`.
- **One commit per change**, each with a message saying what changes and why. Stage per logical change, hunk by hunk when one file holds several changes.
- Leave finished branches unmerged for review. Never push or merge unless asked.
- Tag each unity editor version change.

## Documentation

- Keep the README up to date and human readable aswell as any agent file (like a CLAUDE.md).
