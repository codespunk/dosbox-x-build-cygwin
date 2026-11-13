# DOSBox-X Cygwin Build Helper

Build native Windows versions of DOSBox-X from Cygwin using MinGW-w64.

## Features

- Supports 32-bit and 64-bit Windows builds.
    
- Automatically selects the appropriate MinGW-w64 cross-compiler.
    
- Checks required Cygwin packages before compilation.
    
- Builds and links bundled static libpng.
    
- Produces Windows executables without requiring `cygwin1.dll`.
    

## Usage

Place `build-cygwin` in the root of the DOSBox-X source tree and make it executable:

```
chmod +x build-cygwin
```

Build for 64-bit Windows (default):

```
./build-cygwin
```

Build for 32-bit Windows:

```
./build-cygwin 32
```

Display usage information:

```
./build-cygwin --help
```

Additional configuration options can be passed to the build script.

## Requirements

- Cygwin
    
- MinGW-w64 toolchain for the target architecture
    
- Required development packages (validated automatically before building)
    

## Notes

This helper is intended for use with DOSBox-X source trees that do not yet include integrated Cygwin/MinGW-w64 support in the main `build` script.