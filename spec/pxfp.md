# pxfp: pixel font package

Version 1.

A pxfp file is a plain zip file that carries one pixel font from one library to another.
It holds files of the library as they are: one [pxfm](pxfm.md) file, the [pxgl](pxgl.md) files it names and the [hex](hex.md) palette files it names.
Nothing else goes into it, and nothing but those files comes out of it when it is imported: no [pxtf](pxtf.md) file, no font JSON, no source file.

The package of the fonts of a whole typeface is a [pxtp](pxtp.md) file.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.

## Example

A font package, as the zip file lists it:

```
Archive:  dina-bold.pxfp
  Length   Date       Time    Name
---------  ---------- -----   ----
     1795  1980-01-01 00:00   f4affe733493ee499b027cc47b07cc1e.pxfm
       14  1980-01-01 00:00   pxgl/00c0f3a1b2c3d4e5f60718293a4b5c6d.pxgl
       14  1980-01-01 00:00   pxgl/01d1e4b2c3d4e5f60718293a4b5c6d7e.pxgl
      …
```

Each file is named by its hash, as in the library, but the glyph files and the palette files are flat: there are no prefix folders.
A font of one colour has no palette files, and then no `palette/` folder.

## Contents

A pxfp file holds exactly:

- at the root, one file `<hash>.pxfm`, and no other file at the root;
- in the folder `pxgl/`, `<hash>.pxgl` for every pxgl file the pxfm file names, and for no other;
- in the folder `palette/`, `<hash>.hex` for every palette file the pxfm file names in the fifth field of its glyph lines, and for no other.

All file names inside the package are of lower-case hashes, followed by the extension of the file's kind.
The hash of a file is the first 32 hex digits of the SHA-256 of its canonical form, as the specification of its kind says.
The name of every file inside the package MUST be the hash of its content, by [pxfm](pxfm.md), [pxgl](pxgl.md) and [hex](hex.md).

The `pxgl/` and `palette/` folders are flat.
They have no prefix folders; the importer places each file at its library path.
A pxfm file that names no glyph file at all is possible, and then the package has no `pxgl/` folder.
A pxfm file of a font of one colour names no palette file, and then the package has no `palette/` folder.

The zip file has no other entries: no other folder, no entry for a folder, no `pxfm/` folder, no pxtf file.
An entry name has no backslash, no leading slash and no `..` part.

A pxfp file has the shape of one font only.
A zip file with a pxtf file at the root, or with a `pxfm/` folder, is a [pxtp](pxtp.md) package and no pxfp file, and a reader of pxfp MUST reject it.

## The name of the package file

The name of the package file itself is free.
The zip file already carries the hashes of everything it holds, so a reader and an importer MUST NOT check or depend on the name or the extension of the file they are given.

A writer SHOULD name it `<family>-<style>.pxfp`, so that people can tell packages apart:

- The family and the style are those of the first typeface variant of the library that names this pxfm file: the pxtf files in ascending order of their hash, then the variants of a file in ascending order of their number.
- The family is the slug of the `NAME` of that typeface, and the style the slug of the `STYLE` of that variant; the slug of a name is its lower-case form with every run of characters other than letters and digits made one hyphen, none at the start or the end.
  A part that is not stated is left out, with its hyphen: `dina-bold.pxfp`.
- If no pxtf file names the pxfm file, or neither is stated, the name is the hash of the pxfm file, `<hash>.pxfp`.
- If a file of that name is there already with other bytes, the writer adds `-` and the first 8 hex digits of the pxfm hash before the extension.

This name is recommended, not required.

## Writing

A writer MUST write the zip file so that the same content always gives the same bytes:

- the entries are in the order of the bytes of their names;
- each entry is compressed with standard deflate, at the highest level;
- the time of each entry is 1980-01-01 00:00:00;
- each entry has the same attributes: created on a Unix system, a regular file with mode 0644;
- there is no comment, no extra field, and no entry for a folder.

The name of the package file does not enter the bytes.

## Reading and importing

A reader MUST check the whole package, by the names and contents of its entries and not by the name of the package file, before it uses a file of it, and reject it if:

- it is not a zip file, or has an entry twice, an entry for a folder, or a name that breaks [Contents](#contents);
- its root is not exactly one file `<hash>.pxfm`, or it has a `pxfm/` folder;
- an entry's name is not a lower-case hash with the right extension, or is not the hash of its content;
- the pxfm file does not pass the strict reader of [pxfm](pxfm.md), read with the glyph files of the package;
- a pxgl file the pxfm file names is missing, or a pxgl file is named by no pxfm file;
- a palette file the pxfm file names is missing, or a palette file is named by no pxfm file.

The hash check is in full: every entry's name is compared with the hash of its canonical content, whatever the file is called.

Importing a package places its files in a library at their library paths:

| In the package | In the library |
| --- | --- |
| `<hash>.pxfm` | `pxfm/<first 2 digits>/<hash>.pxfm` |
| `pxgl/<hash>.pxgl` | `pxgl/<first 2 digits>/<hash>.pxgl` |
| `palette/<hash>.hex` | `palette/<first 2 digits>/<hash>.hex` |

The package MUST be checked in full, as above, before any file is placed.
A file that is in the library already with the same bytes is skipped.
If a file is there with other bytes, the importer MUST place nothing and report an error.

An imported package brings only these files.
It brings no pxtf file, no font JSON and no source file.
Nothing deletes files by itself, so what the package brought stays until the owner removes it with `python -m pixelglyphlib.remove`.

## Reference implementation

- `pixelglyphlib/pxfp.py`: `write` makes the package of a pxfm file of a library, `read` reads and checks a package, `unpack` imports it; `pixelglyphlib/pkgbase.py` holds what pxfp and pxtp share.
- `python -m pixelglyphlib.package export pxfm <pxfm hash>... [--out DIR] [--root DIR]` writes `.pxfp` files.
- `python -m pixelglyphlib.package import <file.pxfp or file.pxtp>... [--root DIR]` imports packages of either kind, by their extension.
- `python -m pixelglyphlib.remove pxfm <hash>... --backup DIR` writes the package of each file before deleting it, so that `import` can bring it back.
