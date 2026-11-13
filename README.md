# DOSBox-X Cygwin Build Helper

Build native 32-bit and 64-bit Windows versions of DOSBox-X from Cygwin using the MinGW-w64 cross-compilation toolchains.

## Features

- Supports 32-bit and 64-bit Windows builds.
- Automatically selects the appropriate MinGW-w64 cross-compiler.
- Checks required Cygwin packages before compilation.
- Automatically patches bundled SDL configuration for compatibility with modern MinGW-w64 compilers.
- Detects previously applied SDL patches to avoid applying them twice.
- Builds and links bundled static libpng.
- Produces Windows executables without requiring `cygwin1.dll`.

## Usage

Place `build-cygwin` in the root of the DOSBox-X source tree and make it executable:

```bash
chmod +x build-cygwin
```

Build for 64-bit Windows (default):

```bash
./build-cygwin
```

Build for 32-bit Windows:

```bash
./build-cygwin 32
```

Display usage information:

```bash
./build-cygwin --help
```

Additional configuration options can be passed to the build script.

## Requirements

- Cygwin
- MinGW-w64 toolchain for the target architecture (`x86_64` or `i686`)
- GNU `patch` utility
- Required development packages (validated automatically before building)

The helper checks for missing or incomplete Cygwin packages before compilation and reports any missing dependencies.

## SDL Compatibility

The helper includes embedded patches for the bundled SDL `configure` and `configure.ac` files.

These patches conditionally enable the obsolete `-mno-cygwin` compiler option only when supported by the selected compiler, allowing modern MinGW-w64 toolchains to build SDL successfully.

The patches are applied automatically when necessary. Previously applied patches are detected and skipped.

If the SDL configuration differs from the expected version, the helper stops with an error rather than applying an incompatible patch.

**Note:** The patches modify the SDL files in the working tree. These changes may appear in `git diff`.

## Notes

- MinGW-w64 supports both 32-bit (`i686`) and 64-bit (`x86_64`) Windows targets, despite its name.
- The resulting Windows executables do not require the Cygwin runtime (`cygwin1.dll`).
- This helper is intended for use with DOSBox-X source trees that do not yet include integrated Cygwin/MinGW-w64 support in the main `build` script.
