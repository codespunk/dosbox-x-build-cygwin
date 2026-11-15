# DOSBox-X Cygwin Build Helper

Build native 32-bit and 64-bit Windows versions of DOSBox-X from Cygwin using MinGW-w64, without requiring `cygwin1.dll` at runtime.

## Features

- Builds 32-bit (`i686-w64-mingw32`) and 64-bit (`x86_64-w64-mingw32`) Windows executables.
- Selects the appropriate MinGW-w64 cross-compiler and configures its build environment.
- Builds incrementally by default when a compatible configuration already exists.
- Runs full setup automatically when configuration files are missing or new configure options are provided.
- Checks required Cygwin packages before full setup.
- Applies the bundled SDL compatibility patch when needed.
- Reuses bundled SDL, SDL_net, and static libpng libraries when available, and builds missing archives automatically.
- Supports rebuilding bundled libraries explicitly and compiling individual DOSBox-X components without linking.
- Can strip debugging information from `src/dosbox-x.exe` using MinGW-w64.
- Supports `clean`, `distclean`, and `mostlyclean` cleanup targets.

## Installation

Place `build-cygwin` in the root of the DOSBox-X source tree and make it executable:

```sh
chmod +x build-cygwin
```

## Usage

```text
Usage: ./build-cygwin [32] [options] [configure options...]
       ./build-cygwin {clean|distclean|mostlyclean}
       ./build-cygwin --help
```

### Default builds

```sh
./build-cygwin       # 64-bit Windows (default)
./build-cygwin 32    # 32-bit Windows
```

When `Makefile` and `config.status` already exist and no additional configure options are supplied, the helper invokes Make directly. It skips Cygwin package preflight checks, Autotools regeneration, and configuration, and rebuilds changed files before relinking when necessary.

If the configuration files are missing, the helper performs full setup, checks dependencies, builds bundled libraries, and invokes the repository's `./build` script. Supplying additional configure options also triggers full setup:

```sh
./build-cygwin --enable-some-feature
```

Use actual options supported by the DOSBox-X configure script. To switch between 32-bit and 64-bit targets, clean the existing build tree first: the helper rejects a configured build for the wrong architecture rather than mixing object files.

### Cleaning the build tree

The helper exposes three standard Automake cleanup targets:

```sh
./build-cygwin clean        # Remove compiled files; retain configuration
./build-cygwin distclean    # Also remove generated configuration files
./build-cygwin mostlyclean  # Remove most compilation outputs
```

`clean` is useful when you want to recompile the project while keeping the
existing `Makefile` and `config.status`. `distclean` is more thorough: the next
normal build will detect the missing configuration and run full setup.
`mostlyclean` performs a less extensive cleanup than `clean`.

Each command requires an existing top-level `Makefile`, must be used alone,
and runs without checking the MinGW-w64 compiler or rebuilding libraries.
Before cleanup, the helper refreshes timestamps on existing generated
configuration files to prevent Automake from unnecessarily rerunning configure.
Cleanup affects files managed by the project's Makefiles; separately built
SDL, SDL_net, and libpng archives may remain. Their presence is checked
again on the next normal build.

### Bundled libraries

Before a normal incremental build, the helper checks for these archives:

| Library | Required archive |
| --- | --- |
| SDL | `vs/sdl/linux-host/lib/libSDL.a` |
| SDL main | `vs/sdl/linux-host/lib/libSDLmain.a` |
| SDL_net | `vs/sdlnet/linux-host/lib/libSDL_net.a` |
| libpng | `vs/libpng/linux-host/lib/libpng.a` |

Existing archives are reused without rebuilding. Missing archives are built automatically; if SDL is missing, SDL_net is rebuilt as well. The helper checks that each required archive was produced after an automatic rebuild. These checks verify **existence**, not whether the libraries are up to date with their sources.

To explicitly rebuild bundled libraries as part of a normal build:

```sh
./build-cygwin --with-sdl                 # SDL and SDL_net, then DOSBox-X
./build-cygwin --with-libpng              # libpng, then DOSBox-X
./build-cygwin --with-sdl --with-libpng   # All three, then DOSBox-X
```

### Component-only builds

Use `--only-NAME` to compile a configured component under `src/` without relinking the final executable:

```sh
./build-cygwin --only-cpu
./build-cygwin --only-dos
./build-cygwin --only-gui
./build-cygwin 32 --only-cpu
```

`NAME` must correspond to a directory containing `src/NAME/Makefile`. Make still checks dependencies and nested directories within the selected component. Component-only builds require an existing configuration matching the requested target architecture.

Bundled libraries can also be rebuilt independently:

```sh
./build-cygwin --only-sdl       # SDL and SDL_net only
./build-cygwin --only-sdlnet    # SDL_net only
./build-cygwin --only-libpng    # Static libpng only
```

These options invoke the bundled library build scripts without building or relinking DOSBox-X. The library scripts may perform their own clean/rebuild steps.

After component-level testing, run `./build-cygwin` to check the full project and perform the final link. Component-only compilation cannot detect every cross-component or linker error.

### Stripping debugging information

Use `--strip` to run the normal build and then remove debugging information from `src/dosbox-x.exe`:

```sh
./build-cygwin --strip
./build-cygwin 32 --strip
./build-cygwin --with-sdl --strip
```

The helper uses the target architecture's MinGW-w64 `strip` utility with `--strip-debug`. `--strip` is **not** a standalone operation on an existing executable: it first follows the normal build path. It cannot be combined with `--only-*` options because component-only builds do not link the final executable.

## Options

```text
  32                 Target 32-bit Windows (i686-w64-mingw32)
                     Default: 64-bit Windows (x86_64-w64-mingw32)
  clean              Remove compiled files while retaining configuration
  distclean          Remove compiled files and generated configuration
  mostlyclean        Remove most compilation outputs
  --strip            Strip debug information from src/dosbox-x.exe
                     Runs after the build
  --with-sdl         Rebuild bundled SDL and SDL_net
  --with-libpng      Rebuild bundled static libpng
  --only-NAME        Compile only src/NAME without linking DOSBox-X
  --only-sdl         Rebuild bundled SDL and SDL_net only
  --only-sdlnet      Rebuild bundled SDL_net only
  --only-libpng      Rebuild bundled static libpng only
  -h, --help         Show usage information and exit
```

Cleanup commands cannot be combined with any other arguments. `--only-*` options cannot be combined with one another, `--with-*`, or `--strip`. Additional configure options are forwarded to the repository's `./build` script and cause full setup.

## Requirements

- Cygwin and the MinGW-w64 toolchain for the selected target architecture.
- Required development packages, checked automatically during full setup.
- A compatible configured build tree for component-only builds.

Run `./build-cygwin --help` for the script's current command-line reference.

## Notes

This helper is intended for DOSBox-X source trees that do not yet have integrated Cygwin/MinGW-w64 support in the main `build` script. It works alongside that script rather than replacing it.

Incremental builds avoid unnecessary library rebuilding, but checking for archive existence does not detect outdated library source files. Use `--with-sdl` or `--with-libpng` when you want to rebuild those dependencies explicitly.
