---
name: code-vein-2-costume-mod-skill
description: Build, validate, and deploy Code Vein 2 costume-replacement mods from source meshes and textures through Unreal Engine IoStore packaging. Use for clothing swaps, character outfit conversions, skeletal-mesh replacement, material/texture remapping, and in-game mod verification; do not use for modifying the base game files or for unrelated Unreal asset work.
---

# Code Vein 2 costume replacement workflow

Use this skill to turn a source outfit into a removable Code Vein 2 mod. Keep the work reproducible, validate each binary boundary, and separate local authoring assets from the release container.

## Non-negotiable constraints

- Treat the installed game as read-only. Extract from it; never patch or overwrite its original containers or loose assets.
- Put only the mod container in `Content/Paks/~mods/<mod-name>/`.
- Keep the `.utoc`, `.ucas`, and `.pak` files in the release directory and give all three the same name. Preserve the valid `.pak` emitted by the packer; do not substitute an empty file.
- Confirm the game is closed before copying or replacing mod containers. Back up the previous mod release outside the game directory.
- Make the mod removable by deleting only its `~mods/<mod-name>/` directory.
- Do not include private paths, hostnames, IP addresses, usernames, credentials, proprietary source assets, or personal screenshots in a published skill or repository.
- Only use source assets and game assets that the user is authorized to modify or redistribute. A skill repository should contain procedures and small neutral fixtures, not extracted game content.

## Workflow

### 1. Define the replacement

Record the source outfit, target Code Vein 2 costume slot, target body type, parts to keep, parts to remove, material slots, and the visual acceptance checklist. Decide whether the target slot already supplies the head, body, hands, or feet; avoid exporting duplicate limbs or a duplicate head.

### 2. Extract and inventory the target slot

Use the project-approved Unreal/IoStore extractor with the game's matching engine version and encryption configuration. Extract to a fresh directory outside the game tree. Inventory:

- skeletal mesh and all LODs;
- reference skeleton, parent indices, local transforms, and bone names;
- sections, material indices, bone maps, vertex/index buffers, UVs, normals, tangents, colors, and skin weights;
- material instances and their texture parameters;
- color, mask, normal/roughness, and specular textures, including virtual-texture metadata;
- related body or skin assets needed to make exposed skin seams match.

Keep a machine-readable dump of the original layout. Refuse to patch if expected offsets, counts, formats, or package paths do not match the inventory.

### 3. Author and fit the source outfit

In Blender or an equivalent DCC:

1. Import the source outfit and source armature.
2. Convert coordinate conventions and scale once, explicitly documenting the conversion.
3. Fit the outfit to the target body in the bind pose.
4. Map source bones to the target reference skeleton while retaining target names, indices, and hierarchy.
5. Remove source head, duplicate body, duplicate arms/legs, unused accessories, and hidden geometry that the target slot already supplies.
6. Preserve intentional exposed skin such as hands, thighs, shoulders, neck, and ankles as skin regions rather than clothing regions.
7. Check feet and toes against the target foot and toe bones. Do not infer the target toe direction from a source helper bone whose tail points in a different axis.
8. Inspect front, back, side, bent-arm, crouched, stride, and foot-ground poses before export.

### 4. Transfer weights and build mesh data

Transfer source weights to target bones, then validate every vertex:

- all influences resolve to the target skeleton;
- no vertex is unweighted;
- at most eight influences are emitted per vertex;
- bone indices are local to the section bone map;
- weights are normalized to the target byte representation and sum to 255;
- triangle indices stay inside their section's vertex range;
- section base vertices and index ranges are consistent;
- positions, normals, tangents, UVs, and colors have the expected counts;
- all numeric values are finite.

For skin/clothing boundaries, use one shared surface or identical seam positions and weights. Do not move adjacent sides of a seam by different offsets. Remove duplicate or nearly coincident surfaces instead of hiding z-fighting with a color change.

### 5. Build the texture atlas and materials

Pack the source outfit textures into the target UV layout. Keep the atlas dimensions and color-space conventions explicit. Re-encode the target texture format and virtual-texture layout rather than copying a source format blindly.

Update the target material instance to reference the new color and auxiliary textures. Neutralize or replace target-only dye masks, normals, roughness, metalness, and specular data that would leak the old outfit's appearance. For exposed skin, use the target skin material and target skin UVs where possible; a pale color painted into a clothing material is not equivalent to the game's skin shader.

Validate that each material slot points to the intended resource and that no old target texture remains unintentionally bound.

### 6. Rebuild the target asset

Patch a copy of the fresh legacy extraction. Rebuild the skeletal-mesh payload and update its serialized size fields using the format-aware writer. Write only the assets needed for the replacement. Mirror the game's package-relative directory layout exactly.

Never deploy an intermediate legacy payload, raw `.zen` file, or a directory with missing package dependencies as if it were a finished mod.

### 7. Package and validate

Convert the prepared legacy tree back to UE5 IoStore with the project-approved packer. Then run:

1. container verification (`retoc verify` or the equivalent);
2. a parser-level check that loads the mod mesh, material, and textures together with the base game;
3. mesh checks for vertices, sections, active bones, indices, weights, UVs, finite positions, and material bindings;
4. texture checks for dimensions, format, mip count, virtual-texture tile/chunk sizes, and decoded sample quality;
5. a ZIP/release integrity check if distributing an archive.

A parser pass proves structural validity only. It does not prove animation, lighting, shader appearance, or seam quality.

### 8. Deploy safely

With the game closed:

```text
Content/Paks/~mods/<mod-name>/<mod-name>.utoc
Content/Paks/~mods/<mod-name>/<mod-name>.ucas
Content/Paks/~mods/<mod-name>/<mod-name>.pak
```

Copy the three files atomically where practical, compare hashes against the staged release, and run the parser check against the deployed directory. Record the release version and backup location without recording private machine details.

### 9. Perform in-game acceptance

Equip the target costume and inspect:

- front, back, and both sides;
- hands, thighs, shoulders, collarbones, neck, waist, hips, knees, ankles, and shoe soles;
- standing, walking, running, jumping, crouching, combat, bent arms, and bent legs;
- skin color and shader response, material seams, clipping, duplicate limbs, and foot/toe orientation;
- reload behavior and whether the original costume returns after mod removal.

Use screenshots or video as evidence. Track each visual defect to geometry, weights, UV/material assignment, shader inputs, or animation rather than applying an unrelated cosmetic patch.

### 10. Roll back

Exit the game, remove only the mod directory, and confirm the target costume returns to its original appearance. Keep the original game containers untouched. If a prior mod release must be restored, copy back its three-file set only while the game is closed.

## Completion standard

Call the mod complete only when all of these are true:

- staged and deployed containers parse successfully;
- mesh, material, and texture checks pass;
- no game files were modified;
- the outfit passes static and animation checks in game;
- exposed skin and seams are visually acceptable;
- removal restores the original costume;
- the release contains no private information or extracted assets that should not be redistributed.

Report structural validation and in-game validation separately; never infer the latter from the former.
