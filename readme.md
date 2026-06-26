# Astro Pipeline Blender Add-on

Astro Pipeline is a Blender add-on built for the Astro MOBA workflow. It converts older Unity/FBX-style hybrid character assets into a safe Blender preview scene, generates procedural locomotion, bakes real Blender Actions, and exports a `.glb` that Godot can use through `AnimationPlayer`.

## Stable Workflow

Use the large STEP buttons in order.

### 1. Analyze

Click `STEP 1: Classify Asset`.

This creates `ASTRO_ASSET_CLASSIFIER` and identifies whether the model is a hybrid Unity character, fully skinned character, rigid/modular character, or unknown asset.

Optional reports include Full Character Diagnostic, Conversion Readiness, and Bind Pose Inspector.

### 2. Convert

Click `STEP 2: Preview Conversion`.

This creates `Astro_Preview`. The source character is not modified.

The preview conversion duplicates the armature and meshes, preserves skinned bind poses, bone-parents high-confidence rigid accessories, and uses keep-transform parenting so rotation and scale are preserved.

### 3. Manual Mapping

If validation reports an unparented rigid mesh, select the preview object, set `Manual Target Bone`, then click `STEP 3: Apply Manual Mapping`.

Example:

```text
Jets_02_Preview -> BackpackSocket
```

Manual overrides are saved inside the `.blend` in `ASTRO_MANUAL_MAPPING_OVERRIDES`.

### 4. Validate

Click `STEP 4: Validate Preview Scene`.

This creates `ASTRO_PREVIEW_VALIDATION`.

The report checks armatures, meshes, skinned meshes, rigid accessories, bone-parented objects, remaining manual items, broken image references, and available Actions.

### 5. Animation Authoring

Use the movement sliders to preview locomotion.

When the movement looks correct, set:

```text
Target Armature: Armature_Preview
Action Name: Run_RifleReady
```

Then click `STEP 5: Bake Action`.

This creates a real Blender Action with keyframes. The bake report is `ASTRO_BAKE_ACTION_REPORT`.

### 6. Preview Previous Baked Actions

Set `Load Action`, then click `Load Action for Preview`.

This assigns an existing Action back onto the configured target armature and adjusts the timeline range. The report is `ASTRO_LOAD_ACTION_REPORT`.

### 7. Export

Click `STEP 6: Export Preview GLB`.

This exports only the `Astro_Preview` collection. The report is `ASTRO_PREVIEW_GLB_EXPORT_REPORT`.

In Godot, the baked Action should appear under the imported scene's `AnimationPlayer`.

```gdscript
@onready var animation_player: AnimationPlayer = $AnimationPlayer

func _ready():
    animation_player.play("Run_RifleReady")
```

The exact node path may differ depending on Godot's imported scene hierarchy.

## Preferences

Buttons:

- Save Pipeline Preferences
- Load Pipeline Preferences
- Preferences Report

Saved preferences are stored in `ASTRO_PIPELINE_PREFERENCES`.

This includes common settings such as base pose, movement type, locomotion sliders, bake target armature, bake action name, manual mapping target, and export paths.

## Important Design Decisions

The original source is not modified. The stable workflow operates on `Astro_Preview`.

Many older Unity assets import with armature scale `0.01`. The stable workflow preserves this scale because normalizing it can break skinned bind poses.

Skinned meshes keep their Armature modifiers and bind pose. Rigid meshes are attached to bones/sockets with keep-transform parenting.

## Optional / Experimental Tools

The panel includes legacy tools such as Prepare Conversion Collection, Experimental Convert Duplicate Scale, and Experimental Rebuild Rig. These are not part of the proven stable workflow.

## Add-on Structure

```text
astro_pipeline/
├── __init__.py
├── constants.py
├── utils.py
├── settings.py
├── preferences.py
├── diagnostics.py
├── converter.py
├── locomotion.py
├── export.py
├── rig.py
├── sockets.py
├── poses.py
└── ui.py
```

## Module Responsibilities

- `ui.py`: sidebar panel and guided STEP workflow.
- `settings.py`: scene-level UI settings.
- `preferences.py`: save/load/report pipeline preferences.
- `diagnostics.py`: asset classifier, diagnostics, preview validation, Codex handoff.
- `converter.py`: preview conversion, rigid mapping, manual overrides, experimental tools.
- `locomotion.py`: locomotion generator, live preview, Bake Action, Load Action for Preview, Action Report.
- `export.py`: Preview GLB export and general GLB export.
- `rig.py`, `sockets.py`, `poses.py`: rig, socket, and pose utilities.
- `utils.py`: shared helper functions.
- `constants.py`: standard bone names, sockets, offsets, and rename maps.

## Codex Handoff

Click `Create Codex Handoff Report`.

This creates `ASTRO_CODEX_HANDOFF`, which summarizes the proven workflow, module responsibilities, assumptions, safe modification rules, and recommended improvements.

## Safe Modification Rules

- Prefer adding new operators over changing proven conversion/bake/export logic.
- Do not normalize `Armature_Preview` scale unless specifically testing scale conversion.
- Do not replace keep-transform bone parenting without testing helmet/rifle rotation.
- Do not remove `fake_user` from baked Actions.
- Keep source objects untouched.
- Use `Astro_Preview` for the production workflow.
- Keep experimental tools separate from STEP workflow buttons.

## Current Known Limitations

- Broken texture references are detected but not repaired automatically yet.
- Scale normalization remains experimental.
- Manual mapping still uses a typed target bone field.
- The locomotion generator is intentionally simple and best suited to low-poly/isometric characters.
- Godot import hierarchy may vary, so the `AnimationPlayer` path may need manual adjustment.

## Recommended Next Improvements

- Add Idle, Walk, Sprint, Attack, Hit, Death, and Victory generators.
- Add dropdowns for manual object and target mapping.
- Add texture relinking and packing tools.
- Add a one-click workflow status report.
- Extract shared mapping code into a dedicated `mapping.py` module.
