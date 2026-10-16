---
name: film-negative
description: Develop scanned or camera-digitized film negatives into approved positive images with darktable-cli and negadoctor. Use for color or black-and-white negative conversion, roll calibration, color-cast diagnosis, preview review, and user-directed delivery while preserving originals and existing edits.
license: MIT
---

# Film negative development

Use darktable's negadoctor workflow to produce natural, scene-aware positives. This is a procedure, not a universal film-stock preset. Do not assume one preset works for every roll, scanner, light source, or scene.

## Dependencies and boundaries

- Require an installed `darktable-cli` with negadoctor support. The darktable GUI is the safest way to author or inspect an edit history.
- No bundled scripts. Task-local helpers are fine but must not replace darktable's color-managed RAW pipeline with a naive inversion.
- Never overwrite originals, existing sidecars, user libraries, or approved deliverables. Work on copies in a separate job directory.
- Confirm which files to process or exclude. Do not share previews or results beyond the user's request.

## 1. Inventory and isolate

1. Note the darktable version, input types, existing sidecars, and embedded color profiles. Keep a clear mapping from each input to its working copy, XMP, and output name; check for duplicate names before rendering.
2. Copy the selected inputs, and any histories you start from, into a private working directory and edit only the copies. Pass the intended XMP to every render explicitly instead of relying on sidecar discovery.
3. Use a task-specific configuration, cache, temporary directory, and library, never the user's normal ones.
4. Check the orientation of every frame, record the intended rotation/flip per frame, and apply it in the edit pipeline rather than rotating an exported JPEG. Ask about ambiguous frames. Keep comparison previews in the agreed scan orientation.

Check the installed CLI, even when following this example:

```sh
darktable-cli --version
darktable-cli --help
darktable-cli --help icc-type
```

Some builds exit nonzero when printing help. Compare flags and profile names with the manual for the installed release; the development manual may describe newer behavior.

## 2. Prepare a verified edit history

`darktable-cli` renders the history in an XMP or library; it has no per-module parameter flags, so do not invent `--negadoctor-*` options.

Prefer authoring the history in an isolated darktable GUI session and applying its XMP with the CLI. To generate or modify XMP with a helper, first check the installed release's negadoctor parameter schema and module version against matching source or documentation and a known-good GUI-generated sidecar. Preserve module order, versions, history, and unrelated settings. Do not patch parameter blobs by guessed offsets or reuse them across versions. If the schema cannot be verified, ask for a GUI-authored sidecar instead of falling back to a crude RGB inversion.

For camera scans, separate scan-light calibration from the original scene's lighting:

- Use a no-film image of the scanning light to establish white balance when available; avoid clipping during capture.
- Use the appropriate standard/enhanced camera matrix as input profile, or a verified scanner/input ICC for scanner files. Do not assign a camera matrix to an already color-managed scan.
- Enable lens correction for camera scans, with the taking lens correctly identified; verify the auto-detected lens, focal length, and aperture rather than assuming. Skip for flatbed scans, which involve no camera lens.
- Follow negadoctor's documented working profile guidance (linear Rec. 2020 RGB, or a suitable film-emulsion ICC). Disable conflicting automatic tone mapping such as base curve or filmic rgb; inspect any other enabled tone mapper rather than stacking it blindly. If using tone equalizer, place it after negadoctor as documented.

## 3. Calibrate the roll, then respect each scene

Follow the module's GUI control order: film stock, film properties, corrections, print properties. Avoid dust, scratches, perforations, and clipped pixels in sampled areas.

1. **Film stock and base (D min):** choose color or black-and-white. For color, measure a clean unexposed film border, or a separate unexposed frame from the same roll under the same scanning conditions. Reuse this measured base only for frames sharing those conditions. If no base reference exists, disclose the uncertainty and request one; never present a guessed stock preset as a measurement.
2. **Density range (D max):** sample exposed image content, excluding the film border. Check highlight clipping and adjust per frame where needed.
3. **Scan exposure bias:** use image content/dark lowlights, excluding the border; inspect shadow clipping.
4. **Color corrections:** correct shadows before highlights, and only when needed. Use reliable neutral references, respecting scene lighting and inherent colors. Without a trustworthy neutral reference, compare alternatives and ask the user rather than forcing neutral balance.
5. **Print properties:** adjust paper black, grade/contrast, gloss/highlight compression, and print exposure. Preserve highlight detail, shadow separation, and scene color relationships without flattening the image.

Roll-level film-base calibration is reusable; neutral references, exposure, and creative tone decisions are not automatically transferable to every scene.

## 4. Render previews and iterate with the user

Render a representative set first, covering the lighting, exposure, color, and tonal range of the roll. Review at useful size and check detail crops. Make labeled negative/positive comparisons with matching frame IDs and the agreed crop and orientation. If the negative view is built outside darktable, convert from its actual source ICC to the display profile; never just label linear RGB as sRGB. Ask when the source profile is unknown.

Preview template (POSIX shell): `INPUT_COPY` and `XMP` are a working input and its verified sidecar, `OUTPUT` is a new preview JPEG, and `JOB` is a job directory with `config`, `cache`, and `tmp` subdirectories. Check the installed release's profile name before using `SRGB`. Final delivery settings are up to the user.

```sh
darktable-cli "$INPUT_COPY" "$XMP" "$OUTPUT" \
  --width 1600 --height 1600 --hq true --upscale false \
  --icc-type SRGB \
  --core --configdir "$JOB/config" --cachedir "$JOB/cache" \
  --tmpdir "$JOB/tmp" --library :memory: \
  --conf plugins/imageio/format/jpeg/quality=95
```

Check the exit status and logs, then inspect the rendered file's dimensions, output profile, clipping, and orientation. A successful exit does not mean the color is right.

Show previews, state any uncertainty, and get the user's approval of the look, frame selection, orientations, and output names **before batch export**.

Approval of a representative set is not approval of the roll. Show a contact sheet or previews of every included frame, take feedback per frame and scene, revise individual or shared adjustments, and re-preview. Keep accepted results and check that shared changes do not regress other frames. Repeat until the user confirms the entire roll, then deliver.

## 5. Deliver as requested

- Export, package, and deliver in the format, color space, resolution, and packaging requested by the user.
  - Example only, not a default: full-size JPEG at quality 95 in sRGB, delivered as a ZIP. Another user might want 16-bit TIFF in Display P3, or reduced-size JPEGs for sharing.
- Use meaningful, correct file and ZIP entry timestamps, honoring the user's timestamp policy or existing appropriate dates. Set ZIP entry timestamps explicitly when needed; never leave an accidental default of 1980.

## Official references

Consult the manual matching the installed release. These are the current development-manual entry points:

- [darktable-cli invocation and export options](https://docs.darktable.org/usermanual/development/en/special-topics/program-invocation/darktable-cli/)
- [Core invocation, isolated paths, and in-memory library](https://docs.darktable.org/usermanual/development/en/special-topics/program-invocation/darktable/)
- [negadoctor preparation and controls](https://docs.darktable.org/usermanual/development/en/module-reference/processing-modules/negadoctor/)
