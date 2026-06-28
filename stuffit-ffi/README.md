# stuffit-ffi

`stuffit-ffi` exposes the `stuffit` Rust crate through a small, versioned C ABI.
It builds both shared and static libraries and ships a C header and pkg-config
metadata.

## Build

```sh
cargo cbuild --release -p stuffit-ffi
```

The resulting library is named `libstuffit_ffi`. The exact shared-library
extension depends on the platform. `cargo-c` also writes the generated
pkg-config metadata into the target directory.

## Install

```sh
cargo cinstall --release -p stuffit-ffi --prefix=/usr/local --destdir=/optional/staging/root
```

This installs the shared and static libraries plus:

```text
include/stuffit_ffi.h       -> ${prefix}/include/stuffit_ffi.h
pkgconfig/stuffit-ffi.pc    -> ${prefix}/lib/pkgconfig/stuffit-ffi.pc
```

For staged packaging, pass a writable `--destdir` and let the package manager
copy that tree into the final prefix.

A downstream project can then use:

```sh
cc example.c $(pkg-config --cflags --libs stuffit-ffi)
```

## Ownership and errors

- Archives and writers are opaque handles freed by their matching `*_free`
  function.
- Entry names and fork buffers are borrowed from an archive and remain valid
  until that archive is freed.
- Unsafe entry paths (absolute paths, traversal components, and backslashes)
  are rejected while parsing and writing.
- Serialized output is owned by the caller and must be released exactly once
  with `stuffit_owned_bytes_free`.
- Error state is thread-local. `stuffit_last_error_code()` gives a stable
  machine-readable category and `stuffit_last_error()` gives diagnostic text.
- No Rust panic is allowed to unwind across the C ABI boundary.
