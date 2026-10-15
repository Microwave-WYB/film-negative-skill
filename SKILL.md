---
name: film-negative
description: Develop scanned or camera-digitized film negatives into approved positive images with darktable-cli and negadoctor. Use for color or black-and-white negative conversion, roll calibration, color-cast diagnosis, preview review, and full-resolution delivery while preserving originals and existing edits.
license: MIT
---

# Film negative development

Use darktable's negadoctor workflow to produce natural, scene-aware positives. This is a procedure, not a universal film-stock preset. Do not assume one Aerocolor recipe works for every roll, scanner, light source, or scene.

## Dependencies and boundaries

- Require an installed `darktable-cli` with negadoctor support. The darktable GUI is the safest way to author or inspect an independent edit history; it may be needed when no verified headless parameter-editing method is available.
- No bundled scripts or Python dependency. Temporary helpers are optional, task-local, and must not replace darktable's color-managed RAW pipeline with a naive inversion.
- Never overwrite originals, existing sidecars, user libraries, or previously approved full-resolution deliverables. Work on copies in a separate job directory; hash originals and existing sidecars before and after the job.
- Ask which files may be processed, which must be excluded, where private outputs belong, and whether previews may be shared. Processing approval is not publication approval.

## 1. Inventory and isolate

1. Record the installed darktable version, input types, frame identifiers, dimensions, existing sidecars, and embedded color profiles. Establish a manifest mapping each included input to its working copy, independent XMP, preview, and final filename. Record exclusions explicitly; detect duplicate names before rendering.
2. Copy selected inputs and any histories needed as a starting point into a private working directory. Preserve user histories separately and edit only the working copies. Explicitly supply the intended independent XMP to every render; do not rely on automatic sidecar discovery.
3. Use task-specific configuration, cache, temporary directory, and library. Do not point an experimental process at the user's normal configuration or database. Avoid concurrent writers to the same library/configuration.
4. Inspect all frame orientations, not just a representative frame. Record the intended rotation/flip per frame. Apply orientation in the RAW/edit pipeline before final rendering, rather than rotating an exported JPEG. Ask about ambiguous orientations. Comparison previews should retain the agreed scan orientation unless the user requests otherwise.

Check the installed CLI, even when following this example:

```sh
darktable-cli --version
darktable-cli --help
darktable-cli --help icc-type
```

Some builds return a nonzero exit status when printing help. Compare supported flags and profile names with the documentation for that installed release; the development manual may describe newer behavior.

## 2. Prepare a verified edit history

`darktable-cli` renders history from an XMP or library; it does not provide arbitrary per-module parameter flags. Do not invent `--negadoctor-*` flags or assume export `--conf` keys edit processing modules.

Prefer creating the independent history in an isolated darktable GUI session, then applying its XMP with the CLI. If modifying XMP through a temporary helper, first inspect the installed release's negadoctor parameter schema, module version, and serialization using matching source/documentation and a known-good GUI-generated sidecar. Preserve module ordering, versions, history, and unrelated settings. Never patch opaque parameter blobs by guessed offsets or copy unverified blobs across versions. If schema verification is unavailable, stop and request a GUI-authored sidecar rather than silently substituting a crude RGB inversion.

For camera scans, separate scan-light calibration from the original scene's lighting:

- Use a no-film image of the scanning light to establish white balance when available; avoid clipping during capture.
- Use the appropriate standard/enhanced camera matrix as input profile, or a verified scanner/input ICC for scanner files. Do not assign a camera matrix to an already color-managed scan.
- Follow negadoctor's documented working profile guidance (linear Rec. 2020 RGB, or a suitable film-emulsion ICC). Disable conflicting automatic tone mapping such as base curve or filmic rgb; inspect any other enabled tone mapper rather than stacking it blindly. If using tone equalizer, place it after negadoctor as documented.

## 3. Calibrate the roll, then respect each scene

Follow the module's GUI control order: film stock, film properties, corrections, print properties. Avoid dust, scratches, perforations, and clipped pixels in sampled areas.

1. **Film stock and base (D min):** choose color or black-and-white. For color, measure a clean unexposed film border, or a separate unexposed frame from the same roll under the same scanning conditions. Reuse this measured base only for frames sharing those conditions. If no base reference exists, disclose the uncertainty and request one; never present a guessed stock preset as a measurement.
2. **Density range (D max):** sample exposed image content, excluding the film border. Check highlight clipping and adjust per frame where needed.
3. **Scan exposure bias:** use image content/dark lowlights, excluding the border; inspect shadow clipping.
4. **Color corrections:** correct shadows before highlights, and only when needed. Use plausible scene-specific neutral gray references, not simply a bright patch. Terracotta, brick, colored glass, painted walls, foliage, sunset light, and warm interiors are not neutral targets. Do not remove deliberate warmth or natural coolness in shadows just to make channel averages equal. When no reliable neutral exists, compare alternatives and ask the user rather than forcing gray-world balance.
5. **Print properties:** adjust paper black, grade/contrast, gloss/highlight compression, and print exposure. Preserve highlight detail and shadow separation without flattening the scene. Assess skin, foliage, sky, saturated surfaces, and mixed light when present.

Roll-level film-base calibration is reusable; neutral references, exposure, and creative tone decisions are not automatically transferable to every scene.

## 4. Render previews and obtain approval

Render a representative set first: include different lighting, exposures, skin tones, saturated objects, and difficult shadows/highlights. Review at useful viewing size and inspect detail crops. Create labeled negative/positive comparisons with matching frame IDs, inclusion count, crop, and agreed orientation. If building the negative view outside darktable, convert from its actual embedded/known source ICC to the display profile; never merely label linear RGB data as sRGB. Ask when the source profile is unknown.

The following POSIX-shell template assumes `INPUT_COPY` and `XMP` identify a working input and verified sidecar; `OUTPUT` is a new preview JPEG filename and `JOB` is a dedicated job directory. Create its `config`, `cache`, and `tmp` subdirectories first. `SRGB` is supported by some releases; verify the installed profile name before use.

```sh
darktable-cli "$INPUT_COPY" "$XMP" "$OUTPUT" \
  --width 1600 --height 1600 --hq true --upscale false \
  --icc-type SRGB \
  --core --configdir "$JOB/config" --cachedir "$JOB/cache" \
  --tmpdir "$JOB/tmp" --library :memory: \
  --conf plugins/imageio/format/jpeg/quality=95
```

The JPEG extension selects the export format. The documented JPEG quality configuration is an export option, not a negadoctor setting. Review all enabled histories/defaults in the isolated configuration. Capture exit status and logs, and inspect the rendered file, dimensions, embedded output profile, clipping, and orientation; a successful process alone does not establish correct color.

Show previews and disclose uncertainties. Obtain explicit approval of the look, frame selection, orientations, full-resolution export, output names, and any delivery metadata policy **before batch exports**. Revise and re-preview if approval conditions change.

## 5. Export and verify delivery

- Render only the approved manifest, using each frame's approved history. For full resolution, use the same verified command with `--width 0 --height 0` (where documented by the installed version). Never upscale a thumbnail into a final deliverable. Confirm native/cropped dimensions and correct pipeline orientation.
- Deliver sRGB JPEGs with the intended embedded profile and agreed quality. Keep private originals and independent edit histories separate. Strip location/private metadata from delivery copies if requested; verify metadata rather than assuming it is absent.
- Match filenames, frame IDs, counts, exclusions, dimensions, profile, and output hashes against the manifest. Inspect every final image for orientation and obvious color/render failures. Recheck original and pre-existing sidecar hashes to demonstrate preservation.
- If making a ZIP, set each entry timestamp explicitly from the agreed source. In particular, a newly constructed `ZipInfo(name)` defaults to 1980; do not let that become an accidental delivery date. A scan's EXIF date is not necessarily the film exposure date. Ask how uncertain dates should be represented; do not invent exposure dates.
- Verify ZIP entry names/count, decompression/CRC, and uncompressed payload hashes. When only repackaging, payload hashes must match the approved full-resolution JPEGs exactly. Never replace approved originals with resized previews or re-encoded files just to repair ZIP metadata.
- Report delivered paths, counts, dimensions/profile checks, preservation checks, and remaining uncertainties. Upload or publish only with separate explicit permission.

## Official references

Consult the manual matching the installed release. These are the current development-manual entry points:

- [darktable-cli invocation and export options](https://docs.darktable.org/usermanual/development/en/special-topics/program-invocation/darktable-cli/)
- [Core invocation, isolated paths, and in-memory library](https://docs.darktable.org/usermanual/development/en/special-topics/program-invocation/darktable/)
- [negadoctor preparation and controls](https://docs.darktable.org/usermanual/development/en/module-reference/processing-modules/negadoctor/)
