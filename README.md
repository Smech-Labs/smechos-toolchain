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

**This release is the compiler only.** Everything the compiler builds
(Mesa, Qt6, KF6, Plasma, etc.) is compiled from source by
`spk-compile.py`'s own phases, the same way the kernel and systemd
already are today — there is no separate pre-built sysroot tarball to
download. A sysroot was built once during verification purely to prove
the toolchain actually works end-to-end; it was never meant to ship.

## What's in this release

- **`smechos-toolchain-x86_64-smechos-linux-gnu.tar.xz`** — the compiler
  itself: binutils 2.43.1, GCC 14.2.0, GDB 16.2, and glibc 2.41's bare
  headers/libs as crosstool-ng's own sysroot. Unpacks to a single
  top-level `x86_64-smechos-linux-gnu/` directory.
- **`SHA256SUMS`** — checksum for the tarball above.

## Versions, exactly

| Component | Version |
|---|---|
| crosstool-ng | 1.27.0 |
| binutils | 2.43.1 |
| GCC | 14.2.0 |
| glibc | 2.41 |
| GDB | 16.2 |
| Linux kernel headers | 6.13 |

## Requirements to actually use this

- **Host**: x86_64 Linux. Built and tested on the same host architecture
  it targets (this is a cross-toolchain by target triplet, not by host
  architecture — it still runs natively on x86_64).
- **Disk**: ~423MB unpacked.
- **No installation step** — this is crosstool-ng's standard relocatable-ish
  layout. Unpack the tarball somewhere and point your build at the
  resulting path directly (see below). The toolchain prefix is marked
  read-only on disk (`CT_PREFIX_DIR_RO=y` in the original build config) —
  keep it that way; don't write into it.

## How to use it

```bash
# Unpack
tar -xJf smechos-toolchain-x86_64-smechos-linux-gnu.tar.xz

# Put the compiler on PATH
export PATH="$PWD/x86_64-smechos-linux-gnu/bin:$PATH"

# Compile against the bare glibc sysroot that ships with the toolchain
x86_64-smechos-linux-gnu-gcc --sysroot="$PWD/x86_64-smechos-linux-gnu/x86_64-smechos-linux-gnu/sysroot" your_file.c -o your_file
```

For CMake-based projects (Qt6/KF6-style), set a real toolchain file
pointing `CMAKE_C_COMPILER`/`CMAKE_CXX_COMPILER` at the
`x86_64-smechos-linux-gnu-gcc`/`-g++` binaries above, and
`CMAKE_SYSROOT` at whatever target sysroot the build you're running
assembles as it goes — in `spk-compile.py`, each phase installs into a
shared target staging tree, same as it does today against the
container's glibc.

## Licensing

Built from unmodified upstream sources. GCC, binutils, and GDB are
GPL-licensed; glibc is LGPL-licensed. This release redistributes
compiled binaries of all of the above, same as any Linux distribution's
own toolchain packages — see each upstream project for full license
text.

## Not included, on purpose

- Mesa, Qt6, KF6, Plasma — built from source by `spk-compile.py`'s own
  phases against this toolchain, not shipped pre-built here.
- Any integration into `spk-compile.py` itself — this toolchain is not
  yet what SmechOS's actual build pipeline uses to produce a shipping
  image. See `SABI.md` §1 in `Smech-Labs/spk-compile` for the current,
  honest status.
