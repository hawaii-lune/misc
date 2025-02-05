# HWI

HWI is the main code container for Hawaii, it can contain Luau files and objects.

It also specifies how to pull other packages out of a master XHWI file, if present.

> **Reference Implementation**
>
> A complete implementation is available in the
> [PackageIO](https://github.com/hawaii-lune/rt/tree/bootstrapper-rc/src/Exports/PackageIO) package.

# Basic Structure

## Custom Types

### STRING

`STRING` in this format refers to a `ULEB128` length-prefixed string. This format assumes all strings are a vector of
`u8` bytes.

### UNIXPATH

This package uses relative path strings using UNIX path syntax, when `UNIXPATH` is present, this refers to a `STRING`
encoded as a UNIX path.

---

All HWI files have the basic structure:

|Name|Type|Description
|-|-|-|
|Header|`3 bytes`|Magic header, always `HWI`|
|Version|`u8`|Versioning byte|
|Name|`STRING`|Name of the HWI object, used by XHWIs and for debugging|
|Options|`u8`|Bitflag for code options|
|Entry|`UNIXPATH`|Local path to the entry script|
|NumDeps|`ULEB128`|Number of dependencies in the package|
|Dependencies|`Dependency[NumDeps]`|Dependencies in the package, can refer outwardly to XHWI if used|
|NumObjs|`ULEB128`|Number of objects in the package|
|Objects|`Object[NumObjs]`|Code or object files (objects) present in the package|
|Signature|`STRING`|Currently disabled until Lune adds its `crypto` lib|


## Header

All HWI files have a 3-byte identifier: `HWI`, followed by a single versioning byte. This should currently always equal
`01`, while a `00` specification does exist, this should not be used as it was for a proof-of-concept build.

## Version

See Header.

## Name

A `STRING` refering to the name of this HWI package. This is used for a parent `XHWI` file to identify the package, and
to aid in debugging when running the HWI package.

## Options

Options is a bit-flag that tells Hawaii how the code, and for compression, all objects, are encoded in the package,
currently, the low 4 bits of this byte signify the following values:

|`Bits 7-4`|`Bit 3`|`Bit 2`|`Bit 1`|`Bit 0`|
|:-:|:-:|:-:|:-:|:-:|
|`Unused`|`Release`|`Native`|`Compiled`|`Compressed`|

### Release

If this bit is set to 1, this targets Release options when compiling the source, this does nothing if the source is
already compiled.

### Native

Enables JIT and Luau codegen when loading the source code.

### Compiled

Signifies if the source in the `/source/` directory is compiled.

### Compressed

Signifies if content in the package, regardless of directory, is compressed using `DEFLATE`.

## Entry

Indicates to the XHWI or debugger where to enter the file, this must be in the `/source/` directory, and must be
embedded within the file.

## Dependencies

Defines the dependencies this package relies on, this is passed to the `import` syntax. A Dependency follows this
struct:

|Name|Type|Description|
|-|-|-|
|Alias|`STRING`|Local Package name as passed to `import`|
|Dependency|`STRING`|The actual name of the dependency|
|Creator|`STRING`|If defined, must be defined alongside version, and defines the dependency as a remote|
|Version|`STRING`|See Creator|

> `Creator` and `Version` must both be defined, or both must be an empty string. This controls if the dependency is
> a remote dependency

Certain packages are defined as `SYSTEM_PACKS` and must be handled differently, these can never be defined as remote,
but can still be defined within the package to set an alias. Right now, the only `SYSTEM_PACK` is `Hawaii.System`, but
this may change in the future.

When running under an XHWI, the dependency must be defined in the parent XHWI, or when debugging, be adjacent to the
package in the same directory.

These can link to each other correctly...

```
└ build
│ └ 📦 Package1.hwi
│ └ 📦 Package2.hwi
│ ...
```

...while these cannot

```
└ build
│ └ include
│ │ └ 📦 Package2.hwi
│ └ 📦 Package1.hwi
│ ...
```

## Objects

Objects are where the main content of a HWI file are stored, they follow this basic struct:

|Name|Type|Description|
|-|-|-|
|Path|`UNIXPATH`|Object path from root of the package|
|Content|`STRING`|Compressed content if `Options.Compressed` is true, or raw content|

While not formally defined, the runtime expects source files to be in `/source/`, and all other objects to be in
`/objects/`.

## Signature

**TODO**