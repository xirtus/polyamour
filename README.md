# Polyamour

Polyamour is a pure-Rust 3D creation suite originally built for [SexOS](https://github.com/xirtus/sexos) and running natively on **macOS** (Cocoa / CoreGraphics), **Linux** (Wayland / X11), and **SexOS**: modelling, sculpting, painting, UV editing, shader and geometry nodes, rigging and animation, physics (rigid bodies, cloth, particles, smoke and liquid), Grease Pencil, a video sequencer, a compositor, and three render engines (a CPU path tracer, a real-time EEVEE-style rasterizer, and a Workbench solid engine).

It reads and writes `.blend`, `.glb`/`.gltf`, `.obj`, `.stl`, `.ply`, and `.poly` files, supports out-of-process Python scripting via a Blender-compatible `import bpy` / `mathutils` / `bmesh` API, and runs capability-confined pure-Rust `.wasm` add-ons in-process via `wasmi`.

Everything is written in Rust and licensed `GPL-3.0-only`.

## Repository Layout

| Path | Description |
| --- | --- |
| `apps/polyamour` | Interactive 3D suite UI, workspaces, editors, operators, Python host, and native `winit` + `softbuffer` macOS/Linux window |
| `crates/sex-3d` | Pure-Rust 3D core: scene graph, half-edge meshes, modifiers, shader/geometry nodes, render engines, physics, `.blend`/glTF/OBJ/STL/PLY I/O, and `polyops 1` engine |
| `crates/polyamour-sdk` | `#![no_std]` pure-Rust SDK for procedural geometry & `wasm32-unknown-unknown` add-ons |
| `crates/polyamour-wasm` | Capability-sandboxed `.wasm` add-on runner (`wasmi` fuel/memory limits + host `SessionLimits` + atomic undoable transactions) |
| `crates/sex-office` / `crates/sex-font` | Pure-Rust software canvas & TrueType glyph rasterizer with bundled Geist and Noto fonts |
| `scripts/polyamour` | `.blend` SDNA generator plus macOS (`package_macos.sh`) and Linux (`package_linux.sh`) bundlers |

## Quick Start (macOS & Linux)

### Prerequisites

* Rust toolchain (`rustup`, stable or nightly)
* **macOS**: Xcode Command Line Tools (`xcode-select --install`)
* **Linux** (Debian/Ubuntu):
  ```sh
  sudo apt-get install pkg-config libx11-dev libxcb1-dev libxkbcommon-dev libwayland-dev
  ```
* Optional: `python3` (3.10+) on `PATH` for live `import bpy` Python Console, Text Editor scripts, and `.py` add-ons (`.wasm` add-ons and `polyops 1` scripts run in-process without Python).

### Run Interactively

```sh
cargo run --release -p polyamour
```

Open a `.blend` / `.glb` / `.obj` / `.poly` scene or run a `.wasm` / `.py` script at launch:

```sh
cargo run --release -p polyamour -- scene.blend
cargo run --release -p polyamour -- --wasm crates/polyamour-wasm/tests/spiral_rust.wasm
cargo run --release -p polyamour -- --python apps/polyamour/python/examples/spiral_staircase.py
```

Headless PNG render (no window opened):

```sh
cargo run --release -p polyamour -- --sculpt-demo --screenshot out.png
```

### Package Native Bundles

* **macOS (`dist/Polyamour.app` + `.zip`)**:
  ```sh
  bash scripts/polyamour/package_macos.sh          # host architecture (arm64 or x86_64)
  bash scripts/polyamour/package_macos.sh --universal
  ```
* **Linux (`dist/polyamour-linux-<arch>.tar.gz`)**:
  ```sh
  bash scripts/polyamour/package_linux.sh
  ```

### Run Tests

```sh
cargo test --release --workspace
```

### Writing a Pure-Rust `.wasm` Add-on

```sh
rustup target add wasm32-unknown-unknown
cargo build --release --target wasm32-unknown-unknown \
  --manifest-path crates/polyamour-wasm/guests/spiral_rust/Cargo.toml
```

Drop any compiled `.wasm` file into `~/.local/share/polyamour/addons/` (or `$XDG_DATA_HOME/polyamour/addons/`) and enable it under **Edit > Preferences > Add-ons** — its declarative `ui_panel` / `ui_prop` controls appear in the 3D View Sidebar (`N`).
