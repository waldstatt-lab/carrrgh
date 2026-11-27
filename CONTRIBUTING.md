# Contributing

Keep changes focused and explain the problem, resulting behavior, and validation
in the pull request. Start with the [architecture documentation](docs/architecture/README.md)
and [architectural decision records](docs/architecture/decisions/README.md).
AI coding agents also follow [AGENTS.md](AGENTS.md).

## Shaping work

Shape substantial features before implementation: describe the problem, rough
solution, risks, and excluded work. Small fixes and documentation corrections do
not need a pitch. Bound each pitch by a useful playable experiment; add an appetite
or deadline only if your project wants one.

Save pitches as `docs/pitches/<topic>/pitch--<slug>.md` using the
[pitch template](docs/pitches/README.md#template), and add them to the pitch index.
The [Car Drive pitch](docs/pitches/car-drive/pitch--car-movements.md)
is a worked example of the sections: Problem, Solution, Security, Rabbit Holes, and
No-Gos. Explain relevant security concerns or why none apply. Keep Excalidraw
breadboards and rough sketches beside the pitch and link them from Solution when
they help explain interactions or layout.

Pitches describe intent. Architecture documentation describes implemented behavior
and conventions; ADRs preserve significant choices and tradeoffs. Update the
relevant documents with implementation changes. Do not treat the broader
[game idea](docs/idea.md) as a commitment to implement every direction it lists.

## Branches and commits

Use short-lived branches and reviewed pull requests. The default branch is `main`;
use the intended integration branch as the PR base. Prefer `feature/<slug>` or
`bugfix/<slug>`, including an issue ID when useful. Link the relevant issue or
pitch and use a draft PR for early feedback.

Use Conventional Commits: `<type>[optional scope][!]: <description>`. Common types
are `feat`, `fix`, `docs`, `refactor`, `test`, `build`, `ci`, and `chore`. Use a concise
description and separate optional bodies and footers with blank lines. Mark
breaking changes with `!` or a `BREAKING CHANGE:` footer and explain migration
requirements. Squash commit messages follow the same convention.

```text
feat(player): add a wall jump
fix(run): keep collected coins after falling out
docs: describe the level map format
```

## Setup and validation

Follow the [README requirements](README.md#requirements) for Rust and native
dependencies. `Cargo.toml` declares the minimum Rust version and Bevy features;
`Cargo.lock` pins dependency resolution; [.mise/config.toml](.mise/config.toml)
pins the toolchain. On x86_64 Linux, [.cargo/config.toml](.cargo/config.toml)
selects Clang and LLD; on NixOS, run the checks inside `nix-shell`
([shell.nix](shell.nix)). Keep dependency or toolchain changes deliberate, change the
Rust version in `Cargo.toml` and `.mise/config.toml` together, and explain such
changes in the PR.

Run from the repository root for code changes:

```sh
mise run check
```

which runs:

```sh
cargo fmt --check
cargo clippy --locked --all-targets -- -D warnings
cargo build --locked
cargo test --locked
```

For changes that touch browser-only code (`cfg(target_arch = "wasm32")`), the web
page or dependencies, also run:

```sh
cargo clippy --locked --target wasm32-unknown-unknown -- -D warnings
mise run web:serve   # builds dist/web and serves http://127.0.0.1:8080
```

CI ([.github/workflows/ci.yml](.github/workflows/ci.yml)) runs `mise run check`
and Clippy for `wasm32` on every pull request. Every push to `main` runs
[.github/workflows/pages.yml](.github/workflows/pages.yml): `mise run web:build`,
then publishing `dist/web` to GitHub Pages at <https://waldstatt-lab.ch/carrrgh/>.
Pages must use "GitHub Actions" as its source in the repository settings. The
workflows have read-only rights by default; only the publish job may write Pages.

Tests include unit tests beside each module and
headless simulation tests in `src/tests.rs`, which step the real plugins at the
fixed timestep. Prefer focused tests of game rules and ECS systems that can run
without a window. Include regression coverage for meaningful bugs; avoid tests that
merely duplicate implementation details. Run a subset with `mise run test <name>`.

For rendering, input, or gameplay changes, also run:

```sh
mise run run
```

Check the affected behavior manually in Car Drive:

- A/D and Left/Right steer continuously and stop on release. Neither road edge
  can be crossed; the camera keeps the full road visible.
- Traffic fronts and direction remain readable. Same-direction cars are caught
  slowly; opposing cars approach faster. Cars arrive individually on random lanes
  at varying intervals while earlier cars remain visible. Check that gaps remain
  usable, including when boosting and releasing boost.
- Traffic sits at varied offsets within its lane. Amber indicators blink on the
  intended side before and during adjacent lane changes, including oncoming cars.
  Waiting on a lane marking must no longer be a safe strategy. Changes must not
  overlap other traffic or close a pickup's reserved clearance.
- The HUD advances a level every 1,000 metres without a cap; speed stops rising
  at level 10 while traffic keeps getting denser. From level 3, lanes carry both
  directions; from level 6, watch oncoming cars blink briefly and swerve out of a
  same-direction car's lane in front of the player. From 4,000 metres,
  check visibly wider, longer trucks, their cargo beds and matching collision boxes.
  Truck lane changes should be slower. Game over freezes signals; restart returns
  to level 1 with no trucks or old lane-change state.
- Space, Up and W boost; Down and S brake to 75% of cruise. Brake wins over
  boost and recharges energy; releasing brake restores cruise immediately.
- In the garage's Key bindings screen, replace Space with B, add an alternative,
  reject a conflict, cancel capture with Escape, remove an alternative, and restore
  defaults. Captured keys must not activate menu actions. Close and reopen the
  game (and reload the browser): settings persist, and hints use the chosen keys.
- While boosting, the gauge drains, and release restores cruising and recharges.
  Depletion ends boost; release is required to boost again.
- Hits cost one heart and show a steady protection outline. One overlapping car
  cannot drain all lives. Boost gives no collision immunity.
- The third hit freezes traffic and distance and shows the result. Steering and
  boost do nothing until restart. R restores three hearts, full energy, initial
  position, zero distance and initial difficulty with a fresh random traffic sequence.
- The left stick or D-pad steers; South boosts; Select restarts. Escape quits
  natively and does nothing outside the garage in the browser.
- Check a resized window for readable HUD and road; note that the fixed view
  stretches at unusual aspect ratios.

For browser changes, check in current desktop Firefox and Chrome with
`mise run web:serve`: the page shows "Loading …", then the game; a click gives the
field the keyboard, and arrows, Space and Tab do not scroll or leave the page.
Create a player, drive, buy, sell and buy back a car, reload: the player, balance,
garage and listings remain, and the storage inspector shows `car-drive/save`.

Record what was checked; if no graphical environment is available, report that
limitation.

For documentation-only changes, verify relative links and claims against source,
then run `git diff --check`; a full Rust build is unnecessary. Report checks
actually run, failures, and skipped coverage in the PR. Avoid unrelated formatting
and changes to generated `target/` contents.

## Architecture decisions

Use the [ADR workflow and index](docs/architecture/decisions/README.md) for lasting
choices such as physics integration, input handling, persisted formats, or
simulation scheduling. Copy the [template](docs/architecture/decisions/_adr-template.md),
explain the alternatives and consequences, and identify how reviewers can detect
drift. Routine implementation details do not need an ADR.
