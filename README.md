# DOSBox-X Cygwin Build Helper

Build native 32-bit and 64-bit Windows versions of DOSBox-X from Cygwin using MinGW-w64, without requiring `cygwin1.dll` at runtime.

## Features

- Builds 32-bit (`i686-w64-mingw32`) and 64-bit (`x86_64-w64-mingw32`) Windows executables.
- Selects the appropriate MinGW-w64 cross-compiler and sets up its build environment.
- Checks required Cygwin packages before full builds.
- Applies an SDL compatibility patch when needed and builds bundled static libpng.
- Supports incremental builds without rerunning configuration or rebuilding bundled libraries by default.
- Supports compiling individual DOSBox-X components without linking the executable.
- Can rebuild bundled SDL, SDL_net, or libpng on request.
- Can strip debugging information from `src/dosbox-x.exe` using MinGW-w64.

## Installation

Place `build-cygwin` in the root of the DOSBox-X source tree and make it executable:

```sh
chmod +x build-cygwin
```

## Usage

```text
Usage: ./build-cygwin [32] [options] [configure options...]
       ./build-cygwin --help
```

### Full builds

```sh
./build-cygwin       # 64-bit Windows (default)
./build-cygwin 32    # 32-bit Windows
```

Full builds check dependencies, prepare the cross-compiler environment, build bundled libraries, and invoke the repository's `./build` script. Additional configuration options are passed through to that script.

### Incremental builds

```sh
./build-cygwin -i
./build-cygwin 32 -i
./build-cygwin --incremental
```

Incremental builds use the existing configured build tree and invoke Make directly. They skip package preflight checks, Autotools regeneration, configuration, and bundled-library rebuilding unless explicitly requested. They rebuild changed files and relink the executable when needed.

To rebuild bundled libraries as part of an incremental build:

```sh
./build-cygwin -i --with-sdl       # SDL and SDL_net, then DOSBox-X
./build-cygwin -i --with-libpng    # libpng, then DOSBox-X
./build-cygwin -i --with-sdl --with-libpng
```

### Component-only builds

Use `--only-NAME` to compile a configured component under `src/` without relinking the final executable:

```sh
./build-cygwin --only-cpu
./build-cygwin --only-dos
./build-cygwin --only-gui
./build-cygwin 32 --only-cpu
```

`NAME` must correspond to a directory with an existing `src/NAME/Makefile`. Component-only builds automatically use incremental mode; `-i` is not required. Make still checks dependencies and subdirectories *within* the selected component.

Bundled libraries can also be rebuilt independently:

```sh
./build-cygwin --only-sdl       # SDL and SDL_net
./build-cygwin --only-sdlnet    # SDL_net only
./build-cygwin --only-libpng    # Static libpng only
```

These library options invoke the bundled library build scripts; they do not compile or relink DOSBox-X itself. Those scripts may perform their own clean/rebuild steps.

After testing a component, run `./build-cygwin -i` to perform the complete incremental compilation and link. Component-only compilation cannot detect every cross-component or final-link error.

### Stripping debugging information

Use `--strip` to remove debugging information from the Windows executable:

```sh
./build-cygwin --strip           # Strip an existing 64-bit executable
./build-cygwin 32 --strip        # Strip an existing 32-bit executable
./build-cygwin -i --strip        # Incremental build, then strip
./build-cygwin 32 -i --strip     # 32-bit incremental build, then strip
```

The executable must exist at `src/dosbox-x.exe`. The helper uses the
architecture-specific MinGW-w64 `strip` utility with `--strip-debug`.
When `--strip` is used alone, no build or configuration is performed.
It can also be added to a full build. It cannot be combined with
`--only-*` because those operations do not link the final executable.

## Options

```text
  32                 Target 32-bit Windows (i686-w64-mingw32).
                     Default: 64-bit Windows (x86_64-w64-mingw32).
  -i, --incremental  Build changed files using the existing configuration.
  --strip            Remove debug information from src/dosbox-x.exe.
  --with-sdl         Also rebuild bundled SDL and SDL_net.
  --with-libpng      Also rebuild bundled static libpng.
  --only-NAME        Compile only src/NAME without linking DOSBox-X.
  --only-sdl         Rebuild bundled SDL and SDL_net only.
  --only-sdlnet      Rebuild bundled SDL_net only.
  --only-libpng      Rebuild bundled static libpng only.
  -h, --help         Show usage information and exit.
```

The `--with-*` options are for incremental builds. `--only-*` options cannot be combined with `--with-*`, `--strip`, or with each other. Additional configure options are accepted only for full builds.

## Requirements

- Cygwin and a MinGW-w64 toolchain for the selected target architecture.
- Required development packages, checked automatically during full builds.
- An already configured build tree for incremental or component-only builds, with a configuration matching the requested architecture.

Run `./build-cygwin --help` for the current command-line reference.

## Notes

This helper is intended for DOSBox-X source trees that do not yet have integrated Cygwin/MinGW-w64 support in the main `build` script. It works alongside that script rather than replacing it.

Component-only builds are intended for quick compile checks during development. Changes to shared headers or interfaces may require other components to be rebuilt, and only a full incremental build verifies the final executable link.
