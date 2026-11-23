# Car Drive

A small 2D arcade driving game built with [Bevy](https://bevy.org) 0.20.
Steer through three lanes of traffic, use boost to cover more distance, and keep
three hearts for as long as possible. It replaces the template's starter platformer.

The [pitch and drawing](docs/pitches/car-drive/pitch--car-movements.md) describe the
first playable experiment; [architecture](docs/architecture/README.md) describes
its implementation. Cars and road are drawn procedurally, with no external assets.

Play it in a desktop browser at <https://waldstatt-lab.github.io/carrrgh/>, or build it
natively as described below.

## How to play

The car drives forward automatically. Same-direction cars are slower; oncoming
cars approach faster. Traffic arrives continuously on random lanes with varying gaps. Boost increases
speed but offers no protection. A hit costs one heart and briefly grants protection,
shown by a gold outline. After three hits the drive ends and displays your distance.

Rarely, a pickup lies on the road. A red heart restores a lost heart. A cyan shield
is kept (at most one) and absorbs your next hit without costing a heart; the HUD
shows it beside your hearts and a bubble surrounds your car.

On first start you create a player: type a name (or keep "Driver") and press
Enter or the controller's South / Start button. Every 10 metres driven earn one
coin. A drive's coins are added to your saved balance when it ends: at game over,
on restart, or when you quit. The balance is stored in `profiles.sqlite3` in your
data directory (on Linux `~/.local/share/car-drive`); set `CAR_DRIVE_DATA_DIR`
to use another directory, for example for play tests.

Your garage starts with a free Opel Corsa. Buy any of ten vehicles, choose a color,
equip tires, and purchase motor upgrades or a carbon body kit. Compare speed,
steering, boost and size before confirming a purchase. The garage remembers your
collection and active car. Selling a spare vehicle pays coins immediately; the
marketplace then owns it and offers it with its upgrades. Keep at least one car
and select another before selling the active one.

In the browser, the save lives in the page's local storage under the item
`car-drive/save`, separate from any native save. Clearing the site data deletes
it, and a damaged save is moved to `car-drive/save.broken`. The browser's market
is the local one; there is no shared market there.

The default market is local. For trading between separate clients, follow the
[private marketplace guide](docs/marketplace.md). It uses a separate authoritative
database, provisioned accounts and verified drives; local saves cannot be imported.

| Action | Keyboard | Controller |
| --- | --- | --- |
| Steer left / right | A / D or Left / Right | Left stick or D-pad |
| Boost (hold) | Space, Arrow Up or W | South (A / Cross) |
| Brake (hold) | Arrow Down or S | – |
| Restart | R or Backspace | Select |
| Garage after game over / start from garage | G | Start |
| Garage menu focus / activate | Up, Down / Enter | D-pad / South |
| Garage section / cancel confirmation | Tab / Escape | Right shoulder / East |
| Quit | Escape | – |

Escape cancels in the garage; use the window close button to quit there. In the
browser, click the playing field to give it the keyboard; Escape does not quit
there, so close the tab instead.
In shared-market mode R/Select submits the current drive; start the next one
from the garage. Online runs end after 20 minutes.

Open **Key bindings** in the garage to change gameplay keys, add or remove
alternatives, or restore defaults. Each action keeps at least one key; conflicting
assignments are rejected. Keys refer to physical positions, with US letter labels.
G and Escape remain reserved. Settings persist on this device, independently of
the player profile and marketplace, in `key-bindings.json` beside the native save
or `car-drive/key-bindings` in browser local storage.

Hold brake to move at 75% of cruising speed; release to return to cruise immediately.
Brake takes priority over boost and recharges energy. Steering remains available.
Release boost to recharge. If the tank empties, release before boosting again.
Every 1,000 metres a new level begins, without an end: your speed increases up to
level ten, and traffic keeps getting denser. Later, lanes carry both directions and
oncoming cars swerve around slower traffic.
Restart resets difficulty and generates fresh traffic. Distance records are not saved. Movement values and traffic
patterns are initial play-test values.

## What's inside

| Path | Purpose |
| --- | --- |
| `src/` | Headless driving simulation and presentation plugins |
| `docs/idea.md` | Game direction |
| `docs/pitches/` | Feature proposals and sketches |
| `docs/architecture/` | Implemented behavior and decision records |
| `.agents/skills/` | Installed Bevy skills; hashes in `skills-lock.json` |
| `shell.nix` | Native libraries for NixOS |
| `.mise/` | Pinned Rust toolchain and development tasks |

## Requirements

- [mise](https://mise.jdx.dev), recommended: `mise install` provides Rust 1.97.1
  with rustfmt, Clippy and the `wasm32-unknown-unknown` target, plus
  `wasm-bindgen`, `wasm-opt` and `miniserve` for the browser build, as pinned in
  [.mise/config.toml](.mise/config.toml).
  Without mise, install Rust 1.97.1 or newer (edition 2024) with rustup.
- Bevy 0.20.1, downloaded automatically by Cargo. `Cargo.lock` pins dependencies.
- Clang and LLD on x86_64 Linux, used by [.cargo/config.toml](.cargo/config.toml)
  for faster linking. Remove that file to use the default linker instead.
- A working Vulkan (Linux, Windows) or Metal (macOS) graphics driver.

### Linux native libraries

Bevy needs ALSA, udev, X11 and Wayland libraries, and pkg-config. On Arch Linux:

```sh
sudo pacman -S --needed base-devel pkgconf alsa-lib systemd \
  libx11 libxcursor libxrandr libxi libxkbcommon wayland clang lld \
  vulkan-icd-loader
```

On Debian or Ubuntu:

```sh
sudo apt install build-essential pkg-config libasound2-dev libudev-dev \
  libx11-dev libxcursor-dev libxrandr-dev libxi-dev libxkbcommon-dev \
  libwayland-dev clang lld
```

On NixOS, adding these packages to your profile usually does not make them
visible to builds or the dynamic loader. If your system configuration does not
expose them to Cargo builds, use the project's [shell.nix](shell.nix); it provides
pkg-config, Clang, LLD, and the libraries, and puts the runtime libraries on
`LD_LIBRARY_PATH`. The Vulkan driver comes from the system
(`hardware.graphics.enable = true`):

```sh
nix-shell          # then: mise run run
nix-shell --run 'mise run check'
```

`nix-shell` is not entered automatically when you open the folder. To load it on
`cd`, install [direnv](https://direnv.net) with
[nix-direnv](https://github.com/nix-community/nix-direnv), add a `.envrc` containing
`use nix`, and run `direnv allow` once.

Install a Vulkan driver for your GPU on every distribution. If audio is routed through PipeWire, keep its ALSA plugin
(`pipewire-alsa`) installed.

See Bevy's [Linux dependency instructions](https://github.com/bevyengine/bevy/blob/latest/docs/linux_dependencies.md)
for other distributions and its [setup guide](https://bevy.org/learn/quick-start/getting-started/setup/).

## Development

Read [CONTRIBUTING.md](CONTRIBUTING.md) for the contribution workflow and validation,
the [architecture documentation](docs/architecture/README.md) for current structure
and conventions, and the [ADRs](docs/architecture/decisions/README.md) for recorded
architectural choices. AI coding agents should start with [AGENTS.md](AGENTS.md).

```sh
mise install     # once: install the pinned toolchain
mise run run     # build and start the game
mise run test    # headless tests
mise run build   # debug build; `mise run build --release` for release
mise run check   # format, Clippy, build, and tests before a PR
mise run web:build  # browser version into dist/web
mise run web:serve  # build it and serve it on http://127.0.0.1:8080
mise tasks       # list every task
```

The browser version is the same crate built for `wasm32-unknown-unknown` with
WebGL2 and a size-optimized `web` profile; [web/index.html](web/index.html) is
the page around it. Pull requests are checked by CI, and every push to `main`
publishes the page to GitHub Pages (see [CONTRIBUTING.md](CONTRIBUTING.md)).

Each task passes extra arguments to Cargo, for example `mise run test boost`. The
tasks are plain scripts in [.mise/tasks/](.mise/tasks/), so the equivalent Cargo
commands also work:

```sh
cargo run --locked
cargo fmt --check
cargo clippy --locked --all-targets -- -D warnings
cargo build --locked
cargo test --locked
```

The first build downloads and compiles Bevy and can take several minutes.
Dependencies are optimized in development builds for usable runtime performance.

## Agent skills

`.agents/skills/` holds Bevy 0.20 skills from
[chrisgliddon/bevy-skills](https://github.com/chrisgliddon/bevy-skills): ECS, input,
cameras, UI, assets, audio, animation, physics, rendering, testing, WebAssembly,
and more, plus `similarity-rs` for finding duplicated Rust code. Start with the
`bevy` router skill. [skills-lock.json](skills-lock.json) records each skill's
source and content hash; update them together.

Codex and other agents that follow the `AGENTS.md` convention read these files
directly. Claude Code reads `CLAUDE.md` and `.claude/skills/` instead; link them to
the shared files once per checkout:

```sh
mise run agents:init-claude
```

This creates `CLAUDE.md -> AGENTS.md` and `.claude/skills -> ../.agents/skills`.
Rerunning it is safe, and it never replaces an existing file or a link that points
elsewhere.
