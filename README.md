# pxfdb

A database of pixel fonts as plain text files.

Every file is named by the hash of its canonical form, so the same glyph, palette, font or typeface always has the same name and is stored once.
The files are text, so they can be read, edited and compared in any editor and kept in git.

The formats are specified in the [spec](spec/) folder.
The reference implementation, which reads fonts of many formats into the database and writes them back, is [pixelglyphlib](https://github.com/SzieberthAdam/pixelglyphlib).
The fonts of a database can be browsed and edited with [pxfe](https://github.com/SzieberthAdam/pxfe), a pixel font editor.

## Layout

```
pxgl/<2>/<hash>.pxgl      glyph bitmaps
palette/<2>/<hash>.hex    colour palettes
pxfm/<2>/<hash>.pxfm      fonts: metrics and glyphs
pxtf/<2>/<hash>.pxtf      typefaces: the fonts of one source file
```

`<hash>` is the first 32 hex digits of the SHA-256 of the file's canonical form, in lower case, and `<2>` its first two digits.

## Formats

| Format | File | Holds |
| --- | --- | --- |
| [pxgl](spec/pxgl.md) | `.pxgl` | the bitmap of one glyph |
| [hex](spec/hex.md) | `.hex` | the colours of a glyph of several colours |
| [pxfm](spec/pxfm.md) | `.pxfm` | a font: its stated metrics, and the code, advance, place, bitmap and palette of each glyph |
| [pxtf](spec/pxtf.md) | `.pxtf` | a typeface: the source file, and what it states about each of its fonts |
| [pxfp](spec/pxfp.md) | `.pxfp` | a zip package of one font with its glyphs and palettes |
| [pxtp](spec/pxtp.md) | `.pxtp` | a zip package of one typeface with its fonts, glyphs and palettes |
| [pxfdb](spec/pxfdb.md) | `.pxfdb` | the whole database in one SQLite file, for queries |

The text files are the master.
The packages carry fonts between databases, and the SQLite file is built from the text files and converts back into them byte for byte.

## A glyph

A pxgl file is the rows of the bitmap, `.` for an empty pixel and `0` for an inked one:

```
..0..
.0.0.
0...0
00000
0...0
```

A glyph of several colours numbers them `0` to `9` and `A` to `Z`, in reading order, and its palette is a hex file of a colour a line.

## A font

A pxfm file is sorted lines of tab-separated fields:

```
ASCENT	6
CAPHEIGHT	5
DESCENT	2
GLYPH	I+DEFA00-000	4	0	0
GLYPH	U+000020-000	4	0	0
GLYPH	U+00002C-000	3	0	-2	41E0BE1B2EF9B46F1F72092B18408CE2
GLYPH	U+000041-000	6	0	0	C0FC54C2EC60CB8617B041D991DA1622
KERNING	U+000041-000	U+00004D-000	-1
LINEGAP	1
XHEIGHT	4
```

A `GLYPH` line has the code, the advance, the offsets x and y of the bitmap from the pen on the baseline, and the hash of the glyph file, which a glyph with no pixels, such as a space, has not.

A code is written `G+CCCCCC-VVV`.

- `U+CCCCCC-VVV` is a Unicode code point.
- `X+CCCCCC-VVV` is a glyph without a code point, numbered in the order of the source file.
- `I+DEFA00-VVV` is the default glyph, drawn for a character the font has no glyph for.

The variant `VVV` is `000` for the glyph itself, and `001` to `FFF` for further glyphs of the same code, such as alternatives or the frames of an animation.

Only the metrics the source of the font states are written, and every number is an integer in pixels.
Names, copyright and the other metadata are in the pxtf file, so two fonts that look and measure the same are one pxfm file.
