# Using GPUI (Community Edition) in your Rust Desktop Application

This document explains how to use the GPUI (Community Edition) Rust UI framework to build desktop applications. It covers dependency options (crates.io vs GitHub fork), minimal example code, platform-specific setup, common features, build and run steps, examples to study in the repo, and troubleshooting tips.

---

## 1. Quick summary

GPUI is a hybrid immediate + retained-mode, GPU-accelerated UI framework for Rust. Typical usage: add the `gpui` crate and the `gpui_platform` crate to your Cargo.toml, pick platform features (Wayland/X11 on Linux, default on macOS/Windows), then create an application with `gpui_platform::application().run(|cx: &mut App| { ... })` and build your UI from elements (div, canvas, uniform_list, etc.).

This repo (gpui-ce/gpui-ce) is a community fork that mirrors upstream Zed GPUI; you can either use a published crate (if available) or depend on this fork directly from GitHub.

---

## 2. Dependency options

Option A — (preferred, if `gpui` is published on crates.io)

Add to your Cargo.toml:

```toml
[dependencies]
gpui = "*"
gpui_platform = { version = "*", features = ["wayland", "x11"] } # example for Linux
```

Option B — use this GitHub fork directly (recommended if you want GPUI-CE features or latest commits)

Use the git dependency and the fork package names preserved in this repo. Example using the gpui crate from this repo:

```toml
[dependencies]
# Use the crate as `gpui` in your code while pointing Cargo to this repository
gpui = { git = "https://github.com/gpui-ce/gpui-ce", package = "gpui-ce", rev = "main" }
# Platform integration crate (fork package name gpui_ce_platform)
gpui_platform = { git = "https://github.com/gpui-ce/gpui-ce", package = "gpui_ce_platform", rev = "main", features = ["wayland", "x11"] }
```

Notes:
- The `package` field selects the published package name from this repository; the left-hand key (gpui, gpui_platform) is the crate name you use in your code.
- Replace `rev = "main"` with a tag or commit SHA for reproducible builds.

---

## 3. Minimal `main.rs` example

This is the smallest runnable example using gpui's platform application:

```rust
use gpui::*;

fn main() {
    gpui_platform::application().run(|cx: &mut App| {
        // Build a simple view tree. `div()` and style helpers come from gpui.
        let root = div()
            .id("root")
            .flex()
            .items_center()
            .gap_2()
            .child("Hello, GPUI-CE!");

        // Attach to the app/window (pattern depends on your view setup)
        cx.set_root(root);
    });
}
```

Notes:
- Examples in this repo show many idiomatic patterns (examples/learn). See crates/gpui/examples/learn for full examples (layout, styling, interactivity, lists, canvas).
- If you depend on the fork with package renames (gpui-ce), `use gpui::*;` still works because the library's crate root is exported as `gpui`.

---

## 4. Platform-specific setup

macOS
- Requires Xcode and command line tools.
- Install Xcode and run `xcode-select --install`.
- GPUI uses Metal on macOS; ensure your Xcode toolchain is configured.

Linux (Ubuntu/Debian example)

Install the system development packages required to build and run GPUI:

```sh
sudo apt-get update
sudo apt-get install -y \
  build-essential pkg-config \
  libxkbcommon-dev libxkbcommon-x11-dev libwayland-dev libx11-dev \
  libxcb-shape0-dev libxcb-xfixes0-dev libxcb-randr0-dev libxcb-xinput-dev \
  libegl1-mesa-dev libgles2-mesa-dev libglib2.0-dev libfontconfig-dev
```

Windows
- Use MSVC Rust toolchain with the Visual Studio C++ build tools and a Windows SDK.
- Ensure the Windows crates/features selected by the workspace are enabled when building on Windows.

General notes
- WGPU (GPU rendering) and system GPU drivers must be available and reasonably recent.
- For text shaping Parley is used — no extra per-platform text backend is usually required beyond the platform crates.

---

## 5. Features and common APIs

- Views and styling: Build UI with `div()`, chaining style methods (`flex()`, `bg()`, `rounded_lg()`, etc.).
- State: `Entity<T>` is the primary state container. Call `cx.notify()` to inform observers about updates.
- Actions and keybinds: Define typed actions with the `actions!` macro and register keybinds with `App::bind_keys`.
- Virtualized lists: `uniform_list(..)` provides efficient large lists.
- Custom drawing: `canvas()` lets you paint directly; implement `Element` for full control.
- Async: GPUI includes an async executor integrated with the platform event loop; see `gpui::executor`.

---

## 6. Building and running

1. Install Rust and basic components:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup component add rustfmt clippy
```

2. Add platform system deps (see section 4).

3. Create a new project and add gpui/gpui_platform dependencies (see section 2).

4. Build and run:

```sh
cargo run
```

If you used the git dependency, Cargo will fetch and build the gpui workspace as needed.

---

## 7. Examples and learning resources in this repo

The repository contains many examples at `crates/gpui/examples/learn` and `examples/legacy`. Useful starting points:

- crates/gpui/examples/learn/layout.rs — layout and flex examples
- crates/gpui/examples/learn/styling.rs — styling and transitions
- crates/gpui/examples/learn/interactive_elements.rs — events and state
- crates/gpui/examples/learn/uniform_list.rs — virtualized list example
- crates/gpui/examples/learn/custom_drawing.rs — canvas drawing

Open these examples and adapt them into your application.

---

## 8. Recommendations & best practices

- Dependency pinning: For production apps, pin gpui to a specific commit or semver tag if using the git dependency. Example: `rev = "<sha>"`.
- Features: Only enable the platform features you need (Wayland vs X11) to keep builds smaller.
- Use the examples as patterns for layout and state management; GPUI encourages composing small elements and using Entities for shared state.
- Test cross-platform builds in CI (Linux host won't validate macOS-specific code and vice versa).

---

## 9. Troubleshooting & common errors

- Missing system libs: On Linux, the missing header or library errors indicate you need the packages listed in section 4.
- macOS build failures: Ensure Xcode CLT and bindings (bindgen) are available when building macOS-only features.
- WGPU/backends: If rendering fails, check GPU drivers and WGPU feature flags that may affect adapter selection.

When stuck, inspect build output and check the repo's README and SETUP.md for platform hints; open an issue or join the GPUI/zed Discord indicated in the README.

---

## 10. Where I looked in this repo

- README.md (overview and short examples)
- crates/gpui/README.md (getting started and platform notes)
- Cargo.toml (workspace layout, feature flags, packaging / package names)
- SETUP.md (platform-specific developer setup)
- crates/* (crate list in workspace; examples referenced in crates/gpui/Cargo.toml)

---

If you want, I can:
- Add a minimal example project (skeleton Cargo project) to this repository configured to depend on gpui-ce and run a small window.
- Produce a binary distribution checklist for releasing a GPUI-based app on each platform.
