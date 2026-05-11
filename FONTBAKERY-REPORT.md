# FontBakery report

This file summarises the current state of `fontbakery check-googlefonts` against
the gftools-built artefacts. The detailed HTML report lives in
[FONTBAKERY-REPORT.html](FONTBAKERY-REPORT.html).

To regenerate:

```sh
fontbakery check-googlefonts --succinct \
  --html FONTBAKERY-REPORT.html \
  'fonts/variable/Fixel[wdth,wght].ttf' \
  fonts/ttf/Fixel-*.ttf fonts/ttf/FixelText-*.ttf
```

## Current status (initial onboarding build)

Across the variable font + 18 statics:

| Severity | Count |
|---|---|
| ERROR | 3 |
| FATAL | 0 |
| FAIL  | 272 |
| WARN  | 337 |
| PASS  | 1507 |

The volume is large because the same failure repeats across 19 files. The
distinct categories are listed below; each must be triaged before the family
can be merged into `google/fonts`.

## Distinct FAIL categories

These are upstream-sources issues that the type designers (see [AUTHORS.txt](AUTHORS.txt))
need to address:

- **`googlefonts/vertical_metrics` / `family/win_ascent_and_descent` / `os2_metrics_match_hhea` / `linegaps`** — OS/2 and hhea metrics are inconsistent and don't follow Google Fonts' rules. Typically resolved by running `gftools fix-vertical-metrics` and committing the fixed values into the UFO `fontinfo.plist`.
- **`use_typo_metrics`** — OS/2 `fsSelection` bit 7 (USE_TYPO_METRICS) must be set.
- **`googlefonts/glyph_coverage`** — missing codepoints from the GF Latin Core glyph set. Designers must add the missing glyphs.
- **`googlefonts/glyphsets/shape_languages`** — language-shaping checks (via shaperglot) report failed languages.
- **`whitespace_glyphs`** — missing NBSP (U+00A0).
- **`case_mapping`** — some lowercase glyphs lack uppercase counterparts (or vice versa).
- **`base_has_width`** — base glyphs with zero advance width.
- **`legacy_accents`** — combining accent glyphs incorrectly marked as bases / with width.
- **`name/trailing_spaces`**, **`googlefonts/name/line_breaks`** — name-table records contain stray whitespace / newlines.
- **`googlefonts/license/OFL_copyright`**, **`googlefonts/name/license`** — copyright string and license name records need the GF-required format.
- **`googlefonts/canonical_filename`** — output filenames need to match the GF naming convention exactly (likely just renaming statics from `Fixel-*.ttf` / `FixelText-*.ttf` to `Fixel[wdth,wght].ttf` + a `static/` folder per GF v2 layout).
- **`googlefonts/STAT/axisregistry`** — STAT axis-value names must match Google's [axis registry](https://github.com/google/fonts/tree/main/axisregistry). Width values "Text" / "Display" probably need to be replaced by registry-approved labels.
- **`googlefonts/vendor_id`** — set a registered 4-character vendor ID in `OS/2` (currently `NONE`/blank).

## Distinct WARN categories (lower priority)

- `interpolation_issues` — sources have masters that don't interpolate cleanly at all axis positions.
- `mandatory_avar_table` — variable font lacks an `avar` table; gftools can generate one.
- `overlapping_path_segments`, `outline_direction` — outline-cleanliness issues to fix in the UFOs.
- `googlefonts/article/images` — GF v3 prefers a long-form `ARTICLE.en_us.html` plus sample images; optional but recommended.
- `googlefonts/metadata/unreachable_subsetting` — some encoded glyphs aren't reachable by GF's subsetter.

## Next steps

1. Fix the upstream-sources issues above in the UFOs (type designers).
2. Re-run `gftools builder sources/config.yaml` and this report.
3. Iterate until FAILs drop to zero (WARNs may remain if justified).
4. File the GF onboarding issue at <https://github.com/google/fonts/issues/new/choose>.
