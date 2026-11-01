The image-codec library
=====================

The image-codec library implements image formats such as PNG, BMP, and PPM in
OCaml, relying on only one external dependency: 'decompress'.

Unix-dependent functionality such as reading or writing to files in the
filesystem are packaged in the `image_codec.unix` findlib module inside this
OPAM package; to use it you need to include `image_codec.unix` specifically
in your project's dependencies, for instance `(libraries image_codec.unix)`
in your Dune file.

Supported image formats:
 - PNG (full implementation of RFC 2083),
 - PPM, PGM, PBM, ... (fully supported),
 - BMP (read and write, 24-bit RGB)
 - JPEG (read and write),
    - Baseline sequential and progressive DCT decoding, 8-bit samples,
      arbitrary chroma subsampling and restart intervals.  Writing produces
      baseline sequential images.  Arithmetic coding, lossless modes, 12-bit
      samples and CMYK are not supported.
 - GIF (read/write, animated),
    - Native implementation in the pure `ImageCodec` module handles
      interlaced images, local colour tables, disposal methods 0-3, and
      comment extensions.

As image-codec only requires `decompress`, it is suitable for compilation to
javascript using `js_of_ocaml`, or inclusion in MirageOS unikernels.

The JPEG codec is pure OCaml and currently slower than libjpeg;
see [JPEG-PERFORMANCE.md](JPEG-PERFORMANCE.md) for measurements, a
breakdown of where the time goes, and what could be done about it.

Dependencies
------------
List of dependencies:
 - OCaml (at least 5.0.0)
 - dune (at least 3.14)
 - decompress (version >= 1.5.3)

Additional packages (for test suite only):
 - Crowbar, Alcotest, afl-persistent

Installation
------------

image-codec is available on `opam` (run `opam install image-codec`). It can also
be installed from source as follows.

```bash
dune build
dune install
```

Quick Start: Reading and Writing Images
---------------------------------------

The `ImageCodec_unix` module provides a simple file-based interface for reading and writing images. Add `image_codec.unix` to your dune dependencies:

```ocaml
(libraries image_codec.unix)
```

### Reading an image

```ocaml
open Image
open ImageCodec_unix

let img = ImageCodec_unix.openfile "input.png"
(* img : Image.image *)
```

The format is detected from the file extension. Supported formats: PNG, JPEG, GIF, BMP, PPM/PGM/PBM.

### Writing an image

```ocaml
open Image
open ImageCodec_unix

let img = Image.create_rgb 256 256 in
(* ... fill img ... *)
ImageCodec_unix.writefile "output.jpg" img
```

### Getting image dimensions without decoding

```ocaml
let width, height = ImageCodec_unix.size "photo.jpg"
```

### Converting between formats

```ocaml
(* Convert PNG to JPEG *)
let img = ImageCodec_unix.openfile "photo.png" in
ImageCodec_unix.writefile "photo.jpg" img
```

### Handling unsupported formats

For formats not natively supported (e.g., TIFF, WebP), `ImageCodec_unix` automatically falls back to ImageMagick's `convert` utility if available:

```ocaml
(* This will use ImageMagick to convert TIFF to PNG internally, then read it *)
let img = ImageCodec_unix.openfile "document.tiff"
```

> **Note**: The native implementations are pure OCaml and have no external dependencies. The ImageMagick fallback is only used for unsupported formats.
