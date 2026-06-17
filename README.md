# undroidwish-x86-ci

Builds the **batteries-included `ebuild` undroidwish** natively on an Intel
(x86_64) macOS GitHub Actions runner, so the binary can be **codesigned and
notarized** (it embeds the Tcl/Tk VFS as a `__TEXT,__zipfs` Mach-O section
instead of an appended zip, which `codesign --strict` / Apple's notary reject).

## Why CI instead of building locally

Cross-compiling this ~60-component AndroWish build to x86_64 on an Apple Silicon
Mac is unreliable: macOS does not propagate the `arch -x86_64` preference to
grandchild processes, so `config.guess` detects `aarch64` deep in the sub-builds,
and some components (e.g. libressl) resolve `clang` via an absolute Xcode path,
bypassing both `CC` and `PATH` compiler shims. A **native Intel runner** makes
`config.guess` genuinely x86_64, so everything builds correctly with no tricks —
the same way the arm64 build "just works" natively on Apple Silicon.

## What it does

1. Clones upstream [`charwliu/androwish`](https://github.com/charwliu/androwish)
   at the pinned commit `f73cb8be`.
2. Applies [`patches/macos.patch`](patches/macos.patch) — the local macOS build
   patches (TCL_UTF_MAX=6, SDL2 configure fix, blt/sdl2tk/tkimg/curl/libressl
   tweaks, and the patched `undroid/build-undroidwish-macosx.sh`).
3. Runs `build-undroidwish-macosx.sh ebuild` from a separate build dir.
4. Verifies the result is x86_64 and carries a `__zipfs` section.
5. Uploads `undroidwish-x86_64` as a workflow artifact.

## Using the artifact

Download `undroidwish-x86_64`, then on a Mac:

```sh
lipo -create undroidwish-x86_64 ~/iwish/build-uw-arm64/undroidwish \
  -output undroidwish-universal
```

Then feed the universal binary into the de1app universal packager
(`make_osx_universal_signed.sh`) to produce a signed + notarized universal
`Decent.app`.

GPL-3, same as AndroWish.
