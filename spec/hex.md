# hex: colour palette

Version 1.

A hex file holds the colours of a [pxgl](pxgl.md) glyph, and nothing else.
It is plain text in the hex palette format of Lospec, so it can be read and edited in any text editor, and it is named by the hash of its canonical form, so the same palette always has the same name.
Which glyph uses which palette is told by the [pxfm](pxfm.md) file that names both.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.

## Example

A palette of a grey, a white and a half-transparent black:

```
808080
FFFFFF
00000080
```

Its hash is `dd6506ef74468c7e3f9441cd0a29915f`, so it is stored as `palette/dd/dd6506ef74468c7e3f9441cd0a29915f.hex`.

Line 1 colours the pixel `0` of a glyph file, line 2 the pixel `1`, line 3 the pixel `2`, and so on.

## Lines

A palette is a sequence of lines, one colour each, ended with a line feed (LF, U+000A):

- A colour is written `RRGGBB`: red, green and blue, each from `00` to `FF`, as sRGB values, opaque.
- Or it is written `RRGGBBAA`, with an alpha `AA` as well, from `00` (transparent) to `FE`.
  The alpha `FF` is never written: an opaque colour has six digits, so that it has one form.
  The red, green and blue are not multiplied by the alpha.
- The hex digits are upper case.
  A lower-case digit is no hex digit of this format.

A hex file MUST meet all of these:

1. It is ASCII: every byte is below 0x80.
2. Every line is a colour as above, with no other character: no `#`, no space, no comment.
3. It has at least one line.
4. It has no empty line.
5. No colour is in it twice, the same red, green, blue and alpha: `FF0000` and `FF0000FF` are the same colour, and the second is not even written.

The file carries no other data: no header, no name, no count.

The line number counts the colours from 0: line 1 is the colour of the pixel `0`, line 36 the colour of the pixel `Z`.
The empty pixel `.` has no line: it is the background, drawn as nothing.
A palette MAY have more than 36 lines, but no glyph reaches beyond its 36th.

## Palette of a glyph

A palette serves a glyph when it has at least as many lines as the glyph uses colours, that is, as the highest pixel number plus one.
It MAY have more, so that one palette can serve several glyphs: a glyph that uses the first three colours takes the first three lines, whatever follows them.
The colours of a glyph are numbered in reading order (see [pxgl](pxgl.md#canonical-colours)), so the first colour of a glyph is the first line of its palette.

## Canonical form

The canonical form of a file is made as that of a pxgl file:

1. The text is split into lines at every CR LF (U+000D U+000A), lone CR (U+000D) and lone LF (U+000A).
2. Spaces (U+0020) and tabs (U+0009) are cut from the end of each line.
3. Empty lines are left out.
4. Each line is ended with LF, the last one too.

A writer MUST write the canonical form.
A reader MUST undo the changes of line ends and trailing blanks of steps 1, 2 and 4 before it checks the lines; a file with an empty line is no hex file even though its canonical form would be one, so the reader rejects it.

## Hash and file name

The hash of a file is the first 32 hex digits of the SHA-256 of its canonical form, that is, the first 128 bits of the digest.
In a file name the hex digits are lower case, and upper case when the hash is written inside another file, as a pxfm file writes it.

The file is named `<hash>.hex`.
In a pixelglyphlib library it lies at `palette/<first 2 digits of the hash>/<hash>.hex`, at the root of the library like the `pxgl/` folder.

Since the name is the hash, two files with the same colours in the same order are one file.
A reader that finds a file by its hash SHOULD check that the hash of its canonical form is its name.

## Line ends in git

Git is to keep the files with LF line ends, so that a checkout on any system gives the canonical bytes.
pixelglyphlib's `.gitattributes` says so:

```
/palette/**/*.hex text eol=lf
```

## Errors

A reader MUST reject a file:

- with a byte of 0x80 or above;
- with a line that is no colour of the form above, such as one with a lower-case digit, with an alpha of `FF`, or with a `#`;
- with no line, or with an empty line;
- with the same colour twice.

## Reference implementation

- `pixelglyphlib/store.py`: `PALETTE_DIR`, `palette_path`, `parse_palette`, `check_palette`, `palette_bytes`, `store_palette`, `canonical_pixels`.
