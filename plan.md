# Migrate the zmk-config-fire behavior into zmk-config-v3 while keeping the new repo's build foundation

This ExecPlan is a living document. The sections `Progress`,
`Surprises & Discoveries`, `Decision Log`, and `Outcomes & Retrospective` must
be kept up to date as work proceeds.

This repository includes
[.agent/PLANS.md](/Users/zilizhang/code/zmk-config-v3/.agent/PLANS.md). This
document must be maintained in accordance with that file.

## Purpose / Big Picture

After this work, this repository will build firmware for the same two setups the
user actually uses, Glove80 and the base repo's Corneish Zen target, but with
the user's old typing behavior from `zmk-config-fire` instead of urob's current
default layout. The migrated firmware must preserve the user's alpha positions,
thumb behavior, limited homerow-mod strategy, combo behavior, and the
Mac-specific shortcuts the user explicitly called out, especially `app_swapper`,
`spotlight`, and `hyper_ret`.

The visible proof is straightforward. A person should be able to build both
boards from this repository, inspect the resulting keymap in `draw/base.svg` or
equivalent generated output, and see the old layout behavior reflected there.
The builds must succeed with the repo's current CI and west setup, without mouse
support, leader sequences, Greek/German helper usage, or the Planck target.

## Progress

- [x] (2026-03-17 09:49Z) Compared `/Users/zilizhang/code/zmk-config-fire`
      against this repository and identified the behavioral differences that
      matter for migration.
- [x] (2026-03-17 09:49Z) Resolved migration scope with the user: keep the base
      repo's Corneish Zen target, drop mouse support, drop leader and
      Greek/German helper features, preserve `app_swapper`, `spotlight`,
      `hyper_ret`, and ignore Planck.
- [x] (2026-03-17 09:49Z) Authored this ExecPlan in
      `/Users/zilizhang/code/zmk-config-v3/plan.md`.
- [x] (2026-03-17 09:59Z) Reworked `config/base.keymap` so its active behavior
      matches the old user config rather than the base repo defaults.
- [x] (2026-03-17 09:59Z) Reworked `config/combos.dtsi` so combo timing and
      combo outputs match the old user config.
- [x] (2026-03-17 09:59Z) Rebuilt `config/corneish_zen.keymap` and
      `config/glove80.keymap` so the physical key positions follow the old user
      setup on the new board targets.
- [x] (2026-03-17 09:59Z) Removed active mouse support and the Planck target
      from the live configuration and build matrix. Unused helper files remain
      on disk but are not part of the active config.
- [x] (2026-03-17 10:00Z) Deleted the unused Planck, leader, and mouse helper
      files and removed the unused leader/unicode module entries from
      `config/west.yml`.
- [ ] Local build validation and keymap rendering remain undone. The user
      explicitly said local builds are not needed, so compilation is deferred
      rather than blocked.

## Surprises & Discoveries

- Observation: The GitHub Actions workflow file itself is not the source of the
  old breakage. `/.github/workflows/build.yml` is identical between the old and
  new repositories. Evidence: Both repositories contain the same single-job
  reusable workflow file:

      name: Build ZMK firmware
      jobs:
        build:
          uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main

- Observation: The old repository contains `config/mouse.dtsi`, but mouse
  support was not actually active in the checked-in old configuration. Evidence:
  `/Users/zilizhang/code/zmk-config-fire/config/base.keymap` includes
  `combos.dtsi` and `extra_keys.h` but not `mouse.dtsi`, and the old `.conf`
  files set combo limits rather than `CONFIG_ZMK_POINTING=y`.

- Observation: The biggest migration risk is not CI. It is the mismatch between
  the old board-shape padding system in `config/extra_keys.h` and the new
  per-board `ZMK_BASE_LAYER(...)` wrappers. Evidence: The old repo uses `X_LT`,
  `X_MH`, `X_RH`, and similar macros to inject board-specific keys, while this
  repo hardcodes board wrappers in `config/corneish_zen.keymap` and
  `config/glove80.keymap`.

- Observation: The local shell in this workspace does not include the expected
  firmware build toolchain. Evidence: `just list` fails because the local `yq`
  does not understand the expression syntax used in the `Justfile`, and
  `just init` fails immediately with `sh: west: command not found`.

## Decision Log

- Decision: Use the base repo's active board target for the Corne side instead
  of recreating the old `nice_nano_v2 + corne_left/right` target. Rationale: The
  user explicitly approved using the base repo target. This keeps the migration
  aligned with the current repo's maintained hardware definitions and CI layout.
  Date/Author: 2026-03-17 / Codex

- Decision: Do not carry mouse support into the migrated config. Rationale: The
  user does not use mouse support, and the old repo did not have it wired into
  the active keymap anyway. Date/Author: 2026-03-17 / Codex

- Decision: Do not carry leader sequences or the old Greek/German helper usage
  into the migrated config. Rationale: The user explicitly said they are not
  needed. The goal is behavioral parity for the user's actual setup, not feature
  accumulation from the base repo. Date/Author: 2026-03-17 / Codex

- Decision: Preserve `app_swapper`, `spotlight`, and `hyper_ret` as first-class
  migration requirements, and treat them as Mac-specific behaviors. Rationale:
  The user explicitly named these as behaviors they actively use, and clarified
  that they are used on macOS rather than Windows. Date/Author: 2026-03-17 /
  Codex

- Decision: Remove Planck from the active build matrix, but do not require
  deleting every Planck-related file during the first migration pass. Rationale:
  The user does not need the Planck target. Removing it from `build.yaml` is
  required for behavioral scope. Deleting dead files is optional cleanup and
  should not block the core migration. Date/Author: 2026-03-17 / Codex

- Decision: Skip local build validation for now. Rationale: The user explicitly
  said local builds are not needed, and the plain shell lacks the expected
  build toolchain. Date/Author: 2026-03-17 / Codex

- Decision: Remove unused helper files and west module entries instead of
  merely leaving them dead in the tree. Rationale: The user explicitly said
  those features are not needed, and deleting them makes the migrated config
  easier to read and reason about. Date/Author: 2026-03-17 / Codex

## Outcomes & Retrospective

The repository has now been migrated at the configuration level and stripped of
the unused Planck, leader, mouse, and unicode-specific active scaffolding, but
the result has not been compiled locally. The most important lesson from the
comparison phase is that this work is best handled as a behavioral port onto
the new repo's structure, not as a selective tweak to the current base keymap.

## Context and Orientation

The current repository at `/Users/zilizhang/code/zmk-config-v3` is urob's newer
ZMK config pinned to `v0.3` in `config/west.yml`. It already has working build
infrastructure, a `Justfile`, and a board matrix in `build.yaml`. Its active
keyboard logic lives mostly in `config/base.keymap`, `config/combos.dtsi`, and
the per-board wrappers `config/corneish_zen.keymap` and `config/glove80.keymap`.

The old repository at `/Users/zilizhang/code/zmk-config-fire` is the source of
truth for the user's intended typing behavior. It defines a different alpha
layout, a different thumb arrangement, a limited homerow-mod strategy, different
combo outputs, and a board-padding system through `config/extra_keys.h`. The old
behavior that must be preserved is represented primarily by
`config/base.keymap`, `config/combos.dtsi`, `config/corne.keymap`, and
`config/glove80.keymap`.

In this repository, `config/base.keymap` is the shared logical layout. The board
wrapper files decide where the shared keys land on each physical keyboard.
`build.yaml` decides which targets CI and local multi-target builds will
compile. `config/*.conf` files provide board-specific feature flags such as
pointing support. `config/west.yml` defines external modules and the pinned ZMK
revision. The `Justfile` supplies the local commands for initializing west,
building targets, and rendering the keymap drawing.

The old and new repositories differ in one especially important way. The old
repo spreads board-specific extra keys across `X_*` macros from
`config/extra_keys.h`, while the new repo expects each board file to define
`ZMK_BASE_LAYER(...)`. The migration therefore needs two levels of translation:
first the logical behavior from old shared layout into new shared layout files,
then the physical placement from old board padding into new board wrappers.

## Plan of Work

Start with `config/base.keymap`. Replace the current base repo behavior with the
user's old behavior, while preserving only the parts of the new repo that are
structural improvements rather than behavioral differences. The final
`config/base.keymap` should no longer include `leader.dtsi`, `mouse.dtsi`, or
unicode behavior includes. It should restore the user's old alpha ordering, the
old layer arrangement, the old limited homerow-mod setup with `hrm_hp`-style
special cases instead of full alpha HRMs, and the Mac-specific behaviors the
user relies on. The file should keep working with the current helper macros and
the current board wrapper mechanism used by this repo.

Rework `config/combos.dtsi` next. Port the old combo inventory and timing values
from `/Users/zilizhang/code/zmk-config-fire/config/combos.dtsi`. This includes
the old fast, slow, and homerow combo timing distinctions where still needed,
the old punctuation and symbol combos, and the old chord placements. If any
combo must be represented differently because the current repo structures board
logic differently, keep the output behavior the same and document the deviation
in the `Decision Log`.

Then rebuild the physical wrappers in `config/corneish_zen.keymap` and
`config/glove80.keymap`. These files must use the new repo's
`ZMK_BASE_LAYER(...)` convention, because that is how this repository maps
logical layers onto specific boards. However, the wrapper contents must reflect
the old user setup. For the Corneish Zen target, the thumbs and outer positions
need to emulate the old Corne behavior as closely as the base repo target
allows. For Glove80, the function row, number row, extra edge keys, navigation
keys, and thumb clusters must preserve the old placement decisions rather than
the current base repo defaults.

After the logic and wrappers are aligned, remove active features that the user
does not want. `build.yaml` must drop the Planck target so that local and CI
builds only cover Corneish Zen and Glove80. `config/corneish_zen.conf` and
`config/glove80.conf` must stop enabling pointing. If `config/leader.dtsi`,
`config/mouse.dtsi`, `config/planck_rev6.keymap`, or `config/planck_rev6.conf`
remain in the repository after the first implementation pass, that is acceptable
as long as they are not referenced by active config and do not affect builds.
Removing unused files can be a follow-up cleanup if it is still desirable after
the migrated builds pass.

Finally, validate the result. Initialize west if needed, build all active
targets, regenerate the drawing if the draw recipe still matches the new shared
layout, and compare the rendered layout against the intended old positions.
Record the actual commands run and their outputs in this plan. If a build
failure reveals an incompatibility between the old behavior and the current
ZMK/module versions, update `Surprises & Discoveries` and `Decision Log` before
proceeding with the fix.

## Concrete Steps

All commands below are run from `/Users/zilizhang/code/zmk-config-v3`.

If the west workspace has not been initialized yet, run:

    just init

Expected result:

    west init -l config
    west update --fetch-opt=--filter=blob:none
    west zephyr-export

After editing `config/base.keymap`, `config/combos.dtsi`,
`config/corneish_zen.keymap`, `config/glove80.keymap`,
`config/corneish_zen.conf`, `config/glove80.conf`, and `build.yaml`, build the
active targets:

    just build corneish_zen
    just build glove80

Expected result:

    Building firmware for corneish_zen_v2_left...
    Building firmware for corneish_zen_v2_right...
    Building firmware for glove80_lh...
    Building firmware for glove80_rh...

The exact artifact names may differ slightly if `build.yaml` gains explicit
artifact names, but the build must produce firmware files under `firmware/` for
both halves of both boards.

If the keymap drawer still reflects the shared base layout accurately after the
migration edits, regenerate the visual reference:

    just draw

Expected result:

    draw/base.svg is regenerated without parser errors

If `just draw` no longer reflects the shared base layout because of
repository-specific drawer assumptions, document that limitation here and
validate behavior through successful firmware builds and direct inspection of
the keymap files instead.

## Validation and Acceptance

Acceptance is behavioral, not just textual.

First, `build.yaml` must only describe the active user targets for this
migration: Corneish Zen left and right, plus Glove80 left and right. Planck must
not be part of the active matrix anymore.

Second, `config/base.keymap` must show the user's old logical layout rather than
the current repo's default layout. A reader should be able to open the base
layer and see the old alpha ordering, the old thumb intentions, and the old
Mac-specific shortcuts such as `app_swapper`, `spotlight`, and `hyper_ret`.

Third, the migrated config must not have active mouse, leader, or unicode
behavior. This means `config/base.keymap` must not include `mouse.dtsi` or
`leader.dtsi`, the board `.conf` files must not enable pointing, and the active
logical layers must not depend on the base repo's mouse layer or leader combos.

Fourth, `just build corneish_zen` and `just build glove80` must both succeed.
The human-verifiable proof is the presence of fresh firmware artifacts in
`firmware/` for all four active halves and the absence of build errors
referencing missing behaviors or modules.

Fifth, the physical wrappers must be plausibly faithful to the old setup. The
easiest proof is a regenerated `draw/base.svg` if the drawer remains useful. If
not, open `config/corneish_zen.keymap` and `config/glove80.keymap` and verify
that the board-specific outer keys and thumb positions reflect the old repo's
placements rather than the current base repo defaults.

## Idempotence and Recovery

The plan is intentionally additive and safe to repeat. Re-running
`just build corneish_zen` or `just build glove80` should only refresh build
outputs under `.build/` and `firmware/`. Re-running `just draw` should only
regenerate drawing artifacts under `draw/`.

The riskiest part of the migration is editing the shared logical layout in
`config/base.keymap`, because a malformed behavior definition can break all
boards at once. Recovery is simple: use Git to inspect the last good state, then
re-apply one logical section at a time and rebuild after each group of edits. If
the migration becomes difficult to debug, temporarily narrow `build.yaml` to a
single target, get that target building, then restore the full four-target
matrix before closing the work.

Unused files should not be deleted early unless they actively confuse the build.
Keeping dead files temporarily is safer than deleting potentially useful
references during the first pass. Cleanup can happen after the migrated config
is proven to build.

## Artifacts and Notes

Important old-source files:

    /Users/zilizhang/code/zmk-config-fire/config/base.keymap
    /Users/zilizhang/code/zmk-config-fire/config/combos.dtsi
    /Users/zilizhang/code/zmk-config-fire/config/corne.keymap
    /Users/zilizhang/code/zmk-config-fire/config/glove80.keymap
    /Users/zilizhang/code/zmk-config-fire/config/extra_keys.h

Important destination files:

    /Users/zilizhang/code/zmk-config-v3/config/base.keymap
    /Users/zilizhang/code/zmk-config-v3/config/combos.dtsi
    /Users/zilizhang/code/zmk-config-v3/config/corneish_zen.keymap
    /Users/zilizhang/code/zmk-config-v3/config/glove80.keymap
    /Users/zilizhang/code/zmk-config-v3/config/corneish_zen.conf
    /Users/zilizhang/code/zmk-config-v3/config/glove80.conf
    /Users/zilizhang/code/zmk-config-v3/build.yaml

Critical behavior anchors from the old config that must not be lost:

    The old base alpha layer from config/base.keymap.
    The old `hrm_hp` and related limited homerow-mod strategy.
    The old `app_swapper`, `spotlight`, and `hyper_ret` behaviors.
    The old combo timings and outputs from config/combos.dtsi.
    The old board-specific key placements for Corne and Glove80.

## Interfaces and Dependencies

The migration continues to depend on the ZMK modules already declared in
`config/west.yml` for current builds. The implementation should continue using
`zmk-helpers`, `zmk-auto-layer`, `zmk-adaptive-key`, and `zmk-tri-state` if
those remain necessary for the restored old behavior. `zmk-tri-state` is
especially important because the old and new configs both use it for
swapper-style behaviors.

No active configuration after the migration should require `config/leader.dtsi`,
the leader behavior module, `config/mouse.dtsi`, or ZMK pointing support. If
`config/west.yml` still lists a now-unused module after the first successful
migration build, that is acceptable temporarily, but the active keymap must not
depend on it.

At the end of implementation, the repository must still expose the same public
build interfaces it has now:

    just init
    just build corneish_zen
    just build glove80
    just draw

The semantic contract of those commands after the migration is that they operate
on the user's migrated layout rather than the base repo default.
