# SmechOS toolchain — x86_64-smechos-linux-gnu

A real, from-source cross-toolchain for SmechOS's own target triplet,
built via [crosstool-ng](https://crosstool-ng.github.io/) 1.27.0. This
replaces the old approach (matching the build container's own glibc to
the target) with a genuinely independent toolchain — see `SABI.md` in
[`Smech-Labs/spk-compile`](https://github.com/Smech-Labs/spk-compile)
for the full declared contract this is part of.

**Status**: verified, working, but not yet wired into `spk-compile.py`'s
production build pipeline. Integration is in progress. Until that lands,
treat SmechOS as "has a working, independently-verified toolchain," not
"ships its own glibc" — see `spk-compile.py`'s own `_bootstrap_glibc_runtime`
for what's actually shipping today.

## What's in this release

- **`smechos-toolchain-x86_64-smechos-linux-gnu.tar.xz`** — the compiler
  itself: binutils 2.43.1, GCC 14.2.0, GDB 16.2, and glibc 2.41's bare
  headers/libs as crosstool-ng's own sysroot. Unpacks to a single
  top-level `x86_64-smechos-linux-gnu/` directory.
- **`smechos-sysroot-x86_64-smechos-linux-gnu.tar.xz`** — the
  accumulated target sysroot: Mesa (real `libGL`/`libEGL`/gallium DRI
  drivers), and Qt 6.10.3's `qtbase` + `qtshadertools` (`qtdeclarative`
  still in progress, not included yet), all built against the toolchain
  above. Unpacks to a single top-level `sysroot/` directory.
- **`SHA256SUMS`** — checksums for both tarballs.

## Versions, exactly

| Component | Version |
|---|---|
| crosstool-ng | 1.27.0 |
| binutils | 2.43.1 |
| GCC | 14.2.0 |
| glibc | 2.41 |
| GDB | 16.2 |
| Linux kernel headers | 6.13 |
| Mesa | (see sysroot `libGL.so`/`libEGL.so` for exact build) |
| Qt | 6.10.3 (`qtbase`, `qtshadertools` only — `qtdeclarative` in progress) |

## Requirements to actually use this

- **Host**: x86_64 Linux. Built and tested on the same host architecture
  it targets (this is a cross-toolchain by target triplet, not by host
  architecture — it still runs natively on x86_64).
- **Disk**: ~1.2GB unpacked (423MB toolchain + 556MB sysroot,
  uncompressed).
- **No installation step** — this is crosstool-ng's standard relocatable-ish
  layout. Unpack both tarballs somewhere and point your build at the
  resulting paths directly (see below). The toolchain prefix is marked
  read-only on disk (`CT_PREFIX_DIR_RO=y` in the original build config) —
  keep it that way; don't write into it.

## How to use it

```bash
# Unpack
tar -xJf smechos-toolchain-x86_64-smechos-linux-gnu.tar.xz
tar -xJf smechos-sysroot-x86_64-smechos-linux-gnu.tar.xz

# Put the compiler on PATH
export PATH="$PWD/x86_64-smechos-linux-gnu/bin:$PATH"

# Compile against the full target sysroot (Mesa/Qt6, not just bare glibc)
x86_64-smechos-linux-gnu-gcc --sysroot="$PWD/sysroot" your_file.c -o your_file

# Or for the bare crosstool-ng-generated sysroot alone (glibc only, no Mesa/Qt6):
x86_64-smechos-linux-gnu-gcc --sysroot="$PWD/x86_64-smechos-linux-gnu/x86_64-smechos-linux-gnu/sysroot" your_file.c -o your_file
```

For CMake-based projects (Qt6/KF6-style), set a real toolchain file
pointing `CMAKE_C_COMPILER`/`CMAKE_CXX_COMPILER` at the
`x86_64-smechos-linux-gnu-gcc`/`-g++` binaries above and
`CMAKE_SYSROOT` at the unpacked `sysroot/` directory.

## Licensing

Built from unmodified upstream sources. GCC, binutils, and GDB are
GPL-licensed; glibc is LGPL-licensed; Mesa and Qt are their own
respective licenses (MIT/Expat and LGPL, broadly). This release
redistributes compiled binaries of all of the above, same as any Linux
distribution's own toolchain packages — see each upstream project for
full license text.

## Not included

- `qtdeclarative` and the rest of KDE Frameworks/Plasma — not yet built
  against this toolchain.
- Any integration into `spk-compile.py` itself — this toolchain is not
  yet what SmechOS's actual build pipeline uses to produce a shipping
  image. See `SABI.md` §1 in `Smech-Labs/spk-compile` for the current,
  honest status.
