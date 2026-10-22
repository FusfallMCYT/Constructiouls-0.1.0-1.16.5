# Constructiouls — Forge 1.16.5

This project is based on the Forge 1.16.5 MDK configuration and preserves the existing OBJ/MTL models, textures, and JSON configuration files.

## Environment
- Minecraft: 1.16.5
- Forge: 36.2.34
- ForgeGradle: 5.1.x (MDK configuration)
- Java target: 8
- Gradle wrapper: 7.3.3

## Build
Run `gradlew.bat build` to build the mod. For development, `gradlew.bat runClient` starts the client.

## Current state
- Main mod entry point is `com.jaxcore.constructiouls.Constructiouls`.
- Existing OBJ/MTL files and config JSON files are preserved.
- This is a clean migration baseline, not yet a feature-complete mod: item/block registration, OBJ model loading/rendering, attachment picking, wire drawing, sag generation, and recoloring still need implementation.


## OBJ support (first implementation step)

- Forge's OBJ loader is registered for the `constructiouls` asset domain on the client.
- `com.jaxcore.constructiouls.model.ObjModelData` reads Wavefront OBJ metadata and finds named helper meshes: `wire_attachment_*`, `wire_start`, and `wire_end`.
- Attachment coordinates are calculated from the unique OBJ vertices referenced by each helper mesh's faces, preserving the model's original coordinates.
- MTL library filename is retained for model/config integration.

This is the loader/metadata foundation. It does not yet register placeable pole/isolator blocks or draw wires, and the helper meshes are not yet filtered out of the rendered model. Those are separate next steps; do not treat this build as the complete in-game renderer.

## Models (OBJ + MTL import)
Poles, insulators and wires are drawn by the mod itself from the OBJ/MTL, so **no blockstate / model JSON is needed**.
Only a small config is required:

`assets/<any namespace>/config/<pole|isolator|wire>/<name>.json` (in the mod jar or in ANY resource pack; F3+T reloads),
or `<game dir>/config/constructiouls/<kind>/<name>.json`.

```json
{
  "model":   "mypack:models/pole/my_pole.obj",
  "texture": "mypack:pole/my_pole",
  "offset":  [0.5, 0.0, 0.5],
  "scale":   1.0,
  "flip_v":  true
}
```
- `model` - location of the .obj (the .mtl next to it is read automatically).
- `texture` - used for materials whose `map_Kd` is `#texture` (or missing); a real `map_Kd` path in the MTL wins.
  Atlas style `ns:folder/name` = `assets/ns/textures/folder/name.png`.
- `offset`/`scale` - applied to raw OBJ coordinates. Blockbench exports models centred on 0, so the default offset
  `[0.5, 0, 0.5]` puts them in the middle of the block. If the model sinks into the ground, raise Y (see pole_2: `0.453188`).
- `flip_v` - set `false` if the texture looks upside down.
- Wire models: `sag_amount` (hang depth = amount x length / 4, always straight down). `lateral_*` keys are ignored.
- Helper objects in the OBJ are never drawn: `wire_attachment_N` (insulator/pole points), `wire_start`, `wire_end`, `wire_center`.

## Tools
- GUI: right-click in the air with any tool, pole or insulator (pole/insulator: also sneak + right-click on a block). Values are changed with `[ < value > ]`.
- Pole / Insulator: place; the selected model is remembered in the item.
- Wire Tool: click insulator A, then insulator B (then C, D... to continue the chain; sneak + click clears the selection).
  A wire joins the points with the same number on both insulators (A.1-B.1). One point may hold two wires
  (previous and next section of a line), so A.1-B.1, B.1-C.1, C.1-D.1 form one continuous line. A second wire
  between the same two insulators automatically uses the next point (new line). Setting `Point`: Auto or 1-6 to force a line.
- Tross Tool: same, always straight.
- Wire Remover: look at a wire and right-click; sneak + right-click on a block removes all of its wires.
- Rotation Tool: right-click a pole/insulator to rotate it by `Step` degrees (sneak = opposite direction). Wires follow.
