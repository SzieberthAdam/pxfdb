# pxtp: pixel typeface package

Version 1.

A pxtp file is a plain zip file that carries a pixel typeface, the fonts of one source file, from one library to another.
It holds files of the library as they are: a [pxtf](pxtf.md) file, the [pxfm](pxfm.md) files it names, and the [pxgl](pxgl.md) files and the [hex](hex.md) palette files those name.
Nothing else goes into it, and nothing but those files comes out of it when it is imported: no font JSON, no source file.

The package of one font alone is a [pxfp](pxfp.md) file.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.

## Example

A typeface package of a source with two fonts that share some glyphs, as the zip file lists it:

```
Archive:  dina.pxtp
  Length   Date       Time    Name
---------  ---------- -----   ----
     2210  1980-01-01 00:00   88dce1a14d4ff3f6ba14e2cad7fea034.pxtf
     1833  1980-01-01 00:00   pxfm/0a5b6c1d2e3f4a5b6c7d8e9f0a1b2c3d.pxfm
     1795  1980-01-01 00:00   pxfm/f4affe733493ee499b027cc47b07cc1e.pxfm
       14  1980-01-01 00:00   pxgl/00c0f3a1b2c3d4e5f60718293a4b5c6d.pxgl
       14  1980-01-01 00:00   pxgl/01d1e4b2c3d4e5f60718293a4b5c6d7e.pxgl
      …
```

Each file is named by its hash, as in the library, but the glyph files are flat: there are no prefix folders.

## Contents

A pxtp file holds exactly:

- at the root, one file `<hash>.pxtf`, and no other file at the root;
- in the folder `pxfm/`, `<hash>.pxfm` for every pxfm file the pxtf file names in its `PXFM` lines, and for no other;
- in the folder `pxgl/`, `<hash>.pxgl` for every pxgl file that those pxfm files name, and for no other;
- in the folder `palette/`, `<hash>.hex` for every palette file that those pxfm files name in the fifth field of their glyph lines, and for no other.

All file names inside the package are of lower-case hashes, followed by the extension of the file's kind.
The hash of a file is the first 32 hex digits of the SHA-256 of its canonical form, as the specification of its kind says.
The name of every file inside the package MUST be the hash of its content: a pxtf file by [pxtf](pxtf.md#hash-and-file-name), a pxfm, pxgl or hex file by [pxfm](pxfm.md), [pxgl](pxgl.md) and [hex](hex.md).

The `pxfm/`, `pxgl/` and `palette/` folders are flat.
They have no prefix folders; the importer places each file at its library path.
A pxfm file that names no glyph file at all is possible, and a package whose pxfm files name none has no `pxgl/` folder.
A package whose pxfm files are all of fonts of one colour has no `palette/` folder.

The zip file has no other entries: no other folder, no entry for a folder, no other file at the root.
An entry name has no backslash, no leading slash and no `..` part.

A pxtp file has the shape of a typeface only.
A zip file with a pxfm file at the root and no pxtf file is a [pxfp](pxfp.md) package and no pxtp file, and a reader of pxtp MUST reject it.

## The name of the package file

The name of the package file itself is free.
The zip file already carries the hashes of everything it holds, so a reader and an importer MUST NOT check or depend on the name or the extension of the file they are given.

A writer SHOULD name it `<family>.pxtp`, after the family:

- The family is the slug of the `FAMILY` of the pxtf file, which is its lower-case form with every run of characters other than letters and digits made one hyphen, none at the start or the end.
- If the variants have different families, that of the first variant (the lowest variant number) is used.
- A pxtf file that states no family gives `typeface`.
- If a file of that name is there already with other bytes, such as the package of another source of the same family, the writer adds `-` and the first 8 hex digits of the pxtf hash before the extension.

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
- its root is not exactly one file `<hash>.pxtf`, or it has no `pxfm/` folder;
- an entry's name is not a lower-case hash with the right extension, or is not the hash of its content;
- the pxtf file does not pass the strict reader of [pxtf](pxtf.md);
- a pxfm file does not pass the strict reader of [pxfm](pxfm.md), read with the glyph files of the package;
- a pxfm file the pxtf file names is missing, or a pxfm file in `pxfm/` is not named by it;
- a pxgl file that a pxfm file names is missing, or a pxgl file is named by no pxfm file;
- a palette file that a pxfm file names is missing, or a palette file is named by no pxfm file.

The hash check is in full: every entry's name is compared with the hash of its canonical content, whatever the package file is called.

Importing a package places its files in a library at their library paths:

| In the package | In the library |
| --- | --- |
| `<hash>.pxtf` | `pxtf/<first 2 digits>/<hash>.pxtf` |
| `pxfm/<hash>.pxfm` | `pxfm/<first 2 digits>/<hash>.pxfm` |
| `pxgl/<hash>.pxgl` | `pxgl/<first 2 digits>/<hash>.pxgl` |
| `palette/<hash>.hex` | `palette/<first 2 digits>/<hash>.hex` |

The package MUST be checked in full, as above, before any file is placed.
A file that is in the library already with the same bytes is skipped.
If a file is there with other bytes, the importer MUST place nothing and report an error.

An imported package brings only these files.
It brings no font JSON and no source file, so the library has no font JSON copy that names the imported pxtf file.
The live database of a library is its pxtf and pxfm files and the pxgl and palette files they name; the font JSONs and source files are not part of it.
Nothing deletes files by itself, so what the package brought stays until the owner removes it with `python -m pixelglyphlib.remove`.

## Reference implementation

- `pixelglyphlib/pxtp.py`: `write` makes the package of a pxtf file of a library, `read` reads and checks a package, `unpack` imports it; `pixelglyphlib/pkgbase.py` holds what pxfp and pxtp share.
- `python -m pixelglyphlib.package export pxtf <pxtf hash or font id>... [--out DIR] [--root DIR]` writes `.pxtp` files.
- `python -m pixelglyphlib.package import <file.pxfp or file.pxtp>... [--root DIR]` imports packages of either kind, by their extension.
- `python -m pixelglyphlib.remove pxtf <hash>... --backup DIR` writes the package of each file before deleting it, so that `import` can bring it back.
