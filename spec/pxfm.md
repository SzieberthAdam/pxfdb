# pxfm: pixel font metrics

Version 3.

A pxfm file tells a pixel font by the metrics its source states and the places and shapes of its glyphs.
The glyph bitmaps are [pxgl](pxgl.md) files, named by their hashes; the pxfm file names them, and is itself named by the hash of its canonical form.
The colours of a font of several colours are [hex](hex.md) palette files, named by their hashes as well.
The same font therefore always gives the same bytes and the same hash, whatever its name, its source file or the format it came in, and that hash names the font.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.
In the examples, the fields are separated by tab characters.

## Example

A small font of a space, a comma and four letters, black on white:

```
ASCENT	6
CAPHEIGHT	5
DESCENT	2
GLYPH	I+DEFA00-000	4	0	0
GLYPH	U+000020-000	4	0	0
GLYPH	U+00002C-000	3	0	-2	41E0BE1B2EF9B46F1F72092B18408CE2
GLYPH	U+000041-000	6	0	0	C0FC54C2EC60CB8617B041D991DA1622
GLYPH	U+000048-000	6	0	0	ABE675527C81C31D04CDA5E2AED7623A
GLYPH	U+00004D-000	6	0	0	7C2001124FDBBCFEE5CBF19CC5BC1911
GLYPH	U+000078-000	6	0	0	F83AD16C225C33BC1D6BFE2178677638
KERNING	U+000041-000	U+00004D-000	-1
LINEGAP	1
XHEIGHT	4
```

Its hash is `3b52125b2e5f6466823baa4fa3186fa0`, so it is stored as `pxfm/3b/3b52125b2e5f6466823baa4fa3186fa0.pxfm`.

The source of this font states an ascent of 6, a descent of 2, a line gap of 1, a cap height of 5 and an x-height of 4, and no more, so there are no other metric lines.
The A is drawn by the glyph file `c0fc54c2ec60cb8617b041d991da1622.pxgl`, moves the pen 6 pixels to the right, and has its bottom-left corner on the baseline at the pen position (offsets `0` and `0`).
The comma lies 2 pixels below the baseline (offsets `0` and `-2`).
The space has no pixels and so no glyph file; it moves the pen 4 pixels.
The line with the code `I+DEFA00-000` is the default glyph, the one drawn for a character the font has no glyph for; in this font it is drawn like the space.
The kerning line moves an M that follows an A 1 pixel to the left.

A font whose source states no ascent and no descent, as an image of glyphs does not, has no `ASCENT` and `DESCENT` lines, and nothing else changes:

```
GLYPH	U+000020-000	4	0	0
GLYPH	U+00002C-000	3	0	-2	41E0BE1B2EF9B46F1F72092B18408CE2
GLYPH	U+000041-000	6	-1	0	C0FC54C2EC60CB8617B041D991DA1622
```

Its hash is `4fba4190f6ba7e10b296c7070234e1ce`.
Its A reaches 1 pixel to the left of the pen position (offset x `-1`), and its advance of 5.5 pixels in the source is written `6`.

## What it holds

The file holds what the source of the font states about the font as a whole, and where each glyph is and how far it moves the pen:

- the vertical metrics: ascent, descent, line gap, cap height, x-height and internal leading;
- the quad width;
- the aspect ratio of a pixel;
- the position and thickness of the underline and of the strike-out;
- whether the font is monospaced;
- whether the font is a symbol font;
- the number of its colours, and the palette of each glyph, when it has more than one colour;
- for each glyph, its code, its advance, the place of its bitmap, the hash of its bitmap and, in a font of several colours, the hash of its palette;
- which glyph is the default glyph;
- the kerning.

A metric appears only when the source of the font states it.
Nothing is measured on the glyphs and nothing is derived from them or from other metrics: a font with no stated cap height has no `CAPHEIGHT` line, even if it has an H.

The one conversion is to the pixels of the font.
A source can draw every pixel of the font as a block of several of its own pixels, as a FON file made for pixels twice as tall as wide repeats every row.
Such a font is read on the coarser grid, so one pixel of the font is that block, and every stated metric is converted to it: a vertical value is divided by the height of the block and a horizontal one by its width, rounded half up, and `RATIO` is multiplied by the width and the height of the block.
A FON file of 96 by 96 dots per inch whose rows come twice thus has `RATIO 1:2`, and its ascent and descent count the doubled rows once.

Names, family, style, copyright, licence, the source file, the format of the source and any other metadata are left out; they belong to the [pxtf](pxtf.md) file.
Two fonts that look and measure the same are one font with one hash.

## Encoding and canonical form

The file is ASCII text: every byte is below 0x80.

Its canonical form is made as that of a pxgl file:

1. The text is split into lines at every CR LF, lone CR and lone LF.
2. Spaces and tabs are cut from the end of each line.
3. Empty lines are left out.
4. Each line is ended with LF, the last one too.

A writer MUST write the file in canonical form, with its lines in canonical order (see [Order](#order)).
A reader MUST make the canonical form of what it reads before it reads the lines, so a file whose line ends or trailing blanks were changed on the way reads the same and keeps its hash.

## Hash and file name

The hash of a file is the first 32 hex digits of the SHA-256 of its canonical form, that is, the first 128 bits of the digest.
In a file name the hex digits are lower case.

The file is named `<hash>.pxfm`.
In a pixelglyphlib library it lies at `pxfm/<first 2 digits of the hash>/<hash>.pxfm`, with the glyph files in `pxgl/` and the palette files in `palette/` at the root beside it, and each copy of a font JSON under `sourcefiles/` names the hash of its font's pxfm file in its `pxfm` field.

The canonical order of the lines is not part of the canonical form: a file with its lines in another order has another hash.
Only a file whose lines are in canonical order names its font; a writer MUST NOT write any other.

## Lines

Every line is a key and its fields, separated by single tab characters (U+0009):

```
KEY	FIELD	FIELD…
```

No field is empty, and no field holds a tab, space or other blank.
The keys are upper case, without an underscore:

| Key | Fields | Count |
| --- | --- | --- |
| `ASCENT` | rows above the baseline | at most one |
| `DESCENT` | rows below the baseline | at most one |
| `LINEGAP` | the line gap | at most one |
| `CAPHEIGHT` | height of the capital letters | at most one |
| `XHEIGHT` | height of the small letters | at most one |
| `INTERNALLEADING` | rows at the top of the box meant for accents | at most one |
| `QUADWIDTH` | the width of the quad | at most one |
| `RATIO` | aspect ratio of a pixel, `A:B` | at most one |
| `UNDERLINEPOSITION` | position of the underline | at most one |
| `UNDERLINETHICKNESS` | thickness of the underline | at most one |
| `STRIKEOUTPOSITION` | position of the strike-out | at most one |
| `STRIKEOUTTHICKNESS` | thickness of the strike-out | at most one |
| `MONOSPACED` | none | at most one |
| `SYMBOLIC` | none | at most one |
| `COLORS` | the number of colours | at most one |
| `GLYPH` | code, advance, offset x, offset y, glyph hash, palette hash | one per code: per code point, glyph without a code point and default glyph, and per variant of each |
| `KERNING` | first code, second code, adjustment | one per pair |

A reader MUST reject a line with any other key (the key `COLOR` of version 1 of this format among them), and a line with too few or too many fields.

### Numbers

An integer is written in decimal, with a minus sign (`-`, U+002D) before it if it is negative, with no plus sign and no leading zeros; zero is `0`.

Every number in a pxfm file is an integer, in pixels.
A value that is a fraction in the source is rounded when it is imported, to the nearest whole number, a half towards the larger number: 13.49 becomes 13, 13.5 becomes 14, −0.5 becomes 0 and −1.5 becomes −1.
That is `floor(x + 0.5)`; it is not the rounding of a half to the even number that Python's `round` does.
Advances are rounded in the same way as every other value, and not up.

### Codes

A glyph is named by a code of twelve characters, `G+CCCCCC-VVV`:

- `G` is the group, the letter `U`, `X` or `I`;
- `CCCCCC`, six upper-case hex digits, is the glyph in its group;
- `VVV`, three upper-case hex digits, is the variant.

| Group | Glyphs | `CCCCCC` |
| --- | --- | --- |
| `U` | the code points of Unicode | the code point, from `000000` to `10FFFF` |
| `X` | the glyphs without a code point, the symbols | a running index, from `000000` |
| `I` | the internal glyphs | a fixed value; for now `DEFA00`, the default glyph, only |

The variant `000` is the glyph itself.
The variants from `001` to `FFF` are further glyphs of the same code, such as alternative glyphs or the frames of an animated glyph.
The file does not tell which: a variant is a glyph line like any other, and what it is used for is up to the program that draws the font.

So U+0041 is `U+000041-000`, the first symbol `X+000000-000` and the default glyph `I+DEFA00-000`.

#### Symbols

A glyph that stands for no code point gets a code of the group `X` instead: the first such glyph `X+000000-000`, the next `X+000001-000`, and so on, each index once, with none left out.
They are numbered in the order the glyphs have in the source file, so the same source always gives the same numbers.
How that order is told depends on the source: the position of the glyph in the file for a BDF, PCF or FON file, its glyph index for a TrueType, OpenType or Type 1 font.
A code of the group `X` names the glyph only within the file; the file means nothing else by it.

#### Internal glyphs

The group `I` holds the glyphs that the format itself gives a meaning to, each with a value `CCCCCC` of its own.
For now there is one: `DEFA00`, the default glyph (see [The default glyph](#the-default-glyph)).
A reader MUST reject a code of the group `I` with another value.

#### The code as a number

Where a code is stored as a number, as in a database, the group is a hex digit in the highest bits, `0` for `U`, `A` for `X` and `F` for `I`, then the six digits of `CCCCCC`, and the variant in the lowest bits: `G+CCCCCC-VVV` is the 40-bit number of the ten hex digits `GCCCCCCVVV`.
U+0041 is thus `0000041000` and the default glyph `FDEFA00000`.
The group in the highest bits keeps the code points apart from the symbols, and the variant in the lowest bits lets a program run through the variants of a glyph one after the other.
The numbers do not sort as the codes of the file do, as `I` sorts first among the letters and `F` last among the digits.

### The baseline and the font box

Every position of the file is counted from the baseline and from the pen position.

The baseline is the horizontal line the letters stand on.
The pen position is where a glyph starts: its glyph moves the pen to the right by its advance when it has been drawn.

The font box starts at the pen position, is as wide as the advance of the glyph and is `ASCENT + DESCENT` rows tall: `ASCENT` rows above the baseline and `DESCENT` rows below it.
A glyph MAY reach out of its box in every direction: to the left of the pen position, to the right of its advance, above the ascent and below the descent.

### Vertical metrics

```
ASCENT	6
DESCENT	2
LINEGAP	1
CAPHEIGHT	5
XHEIGHT	4
INTERNALLEADING	1
```

- `ASCENT` is the number of rows above the baseline and `DESCENT` the number of rows below it; neither is negative.
- `LINEGAP` is the gap between two lines of text, in rows, over and above `ASCENT + DESCENT`.
- `CAPHEIGHT` and `XHEIGHT` are the heights of the capital and of the small letters above the baseline.
- `INTERNALLEADING` is the number of rows at the top of the font box that are meant for accents, and are counted in the ascent.

A file has each of these only if the source of its font states it.
A font with no `ASCENT` and no `DESCENT` has no font box of its own: its glyphs lie where their offsets put them, and a program that needs a box takes one from the glyphs.

### QUADWIDTH

```
QUADWIDTH	6
```

`QUADWIDTH` is the width of the quad, as the source of the font states it (the `QUAD_WIDTH` property of a BDF or PCF font), whatever its relation to the glyph of M.

### RATIO

```
RATIO	1:2
```

`RATIO` is the aspect ratio of a pixel: `A:B`, `A` its horizontal and `B` its vertical part.
It comes from the resolutions the source states, such as the horizontal and vertical resolution of a FON file, multiplied by the block a pixel of the font is made of (see [What it holds](#what-it-holds)).
Both are positive integers with no common factor greater than 1, so 96 by 96 dots per inch is `1:1`; a reader MUST reject any other form.

### Underline and strike-out

```
UNDERLINEPOSITION	-2
UNDERLINETHICKNESS	1
STRIKEOUTPOSITION	3
STRIKEOUTTHICKNESS	1
```

The position is that of the top pixel row of the line, counted from the baseline as the offsets of the glyphs are: positive above the baseline, negative below.
The thickness is its number of pixel rows.

### MONOSPACED

```
MONOSPACED
```

`MONOSPACED` is a key alone on its line, with no field and no tab after it.
It is written when the source of the font says that the font is monospaced, whatever the glyphs show: a font that says so and has glyphs of several advances has it, and a font of one advance that does not say so has not.
A font with this key SHOULD be made as a monospaced font.
A reader MUST reject a `MONOSPACED` line with a field.

### SYMBOLIC

```
SYMBOLIC
```

`SYMBOLIC` is a key alone on its line, as `MONOSPACED` is.
It is written when the source of the font says that the font is a symbol font, whose code points are those of a private encoding and not of Unicode characters: a TrueType or OpenType font with a cmap subtable of platform 3, encoding 0 (Windows Symbol), a FON file whose `dfCharSet` is 2, a BDF or PCF font whose `CHARSET_REGISTRY` and `CHARSET_ENCODING` together end in `fontspecific` in any case (as `adobe-fontspecific`), a Type 1 font whose `Encoding` is no `StandardEncoding`.
Nothing else says it: not the code pages of an OS/2 table, not PANOSE, not the glyphs.
A reader MUST reject a `SYMBOLIC` line with a field.

### GLYPH

```
GLYPH	U+000041-000	6	0	0	C0FC54C2EC60CB8617B041D991DA1622
GLYPH	U+000020-000	4	0	0
```

A glyph line has five fields, or four for a glyph with no inked pixels; in a font of several colours (see [COLORS](#colors)) it has six fields, or four for a glyph with no inked pixels:

1. The code (see [Codes](#codes)): that of the code point, or the code of the group `X` of a glyph that has none, or `I+DEFA00-000` for the default glyph.
2. The advance: how far the glyph moves the pen to the right, in whole pixels.
3. The offset x of the bitmap.
4. The offset y of the bitmap.
5. The hash of the glyph's [pxgl](pxgl.md) file, in upper case.
   Its width and height are those of that file.
6. Only in a font of several colours, after the hash of the glyph file: the hash of the glyph's palette, a [hex](hex.md) file, in upper case.

The hashes come after the numbers, so a glyph with no inked pixels, which has no hash, ends its line with its offsets.

The offsets `0` and `0` put the bottom-left corner of the bitmap on the baseline, at the pen position.
A positive x moves the bitmap to the right, a negative x to the left.
A positive y lifts it above the baseline, a negative y moves it below the baseline.
So a glyph whose bitmap is 5 rows tall and stands on the baseline has y 0, and one that has 2 rows below the baseline and the rest above has y −2.

The bitmap MAY reach out of the font box and past the advance; there is no limit.

A glyph with no inked pixels, such as a space, has no glyph file: its line has no fifth field (and no sixth), and both its offsets are `0`.

A glyph of several code points has one line for each, the same but for the code point.
A code SHOULD have one glyph line only.
The glyphs without a code point are one line each, with their own codes, so two that are alike in every other field are still two lines.
Their codes are those of [Symbols](#symbols), from `X+000000-000`, as many as there are such glyphs, each once.

The glyph's bitmap SHOULD be cropped to its inked pixels (see the [pxgl specification](pxgl.md#cropping)), so that the same shape at the same place always gives the same line.

#### The default glyph

```
GLYPH	I+DEFA00-000	4	0	0
```

The default glyph is the one drawn for a character the font has no glyph for.
It has the code `I+DEFA00-000`, an internal glyph, which is no code point and no symbol.
Its variants, as those of any other glyph, have the codes from `I+DEFA00-001` to `I+DEFA00-FFF`.
Its line is a glyph line like any other: it may have any advance, any offsets, and any glyph file or none.
There is at most one line of the code `I+DEFA00-000`.

A source may state its default character as one of its other glyphs (the `dfDefaultChar` of a FON file, the `DEFAULT_CHAR` of a BDF or PCF font).
The importer then writes that glyph once more with the code `I+DEFA00-000`, and the line with its code point stays.
Nothing ties the two lines together, and a reader MUST NOT require that the line of the code `I+DEFA00-000` matches any other line.

A `KERNING` line may name the code `I+DEFA00-000`, as it names any code that a glyph line has.

### COLORS

```
COLORS	2
GLYPH	U+000041-000	6	0	0	3AD25AE0043EDDCC6BC11E8794FCCDD3	EBBBA94934DA6897D86C5E8EA2AC5409
```

A font of one colour is the usual case, and it records nothing of its colour: it has no `COLORS` line and its glyph lines have no palette field.
Its glyph files use the pixel `0` and the empty pixel only, the pixels are black, and the background is transparent or not specified: what the pixels of a source look like is not part of the font.
A font whose pixels were red is the same font as one whose pixels were black, and has the same hash.

A font of more than one colour is a colour font.
It has a line `COLORS` and the number of its colours, an integer of at least 2.
Then every glyph line that has a glyph file has a sixth field, the hash of the glyph's palette (see [hex](hex.md)); a glyph line with no glyph file has four fields as ever.
A font with no `COLORS` line has no palette field in any glyph line.

The glyph files of a colour font are canonical (see [pxgl](pxgl.md#canonical-colours)): the colours are numbered in reading order, and a glyph that uses *k* colours uses the pixels `0` to *k* − 1 of the first *k* lines of its palette.
The palette MAY have more lines, so one palette can serve several glyphs, but never fewer lines than the glyph uses colours.

`COLORS` is the number of different colours the glyph lines refer to, counted like this:

1. For each glyph line that has a palette, take the first *k* lines of the palette, *k* being the number of colours the line's glyph file uses (its highest pixel number plus one).
   The lines of the palette beyond them are not counted, as no glyph of the font refers to them.
2. Put all the colours so taken, of all the glyph lines, in one set.
   Two colours are the same when their red, green, blue and alpha are: `FF0000` and `FF0000FF` are one colour.
3. `COLORS` is the number of colours in the set.
   The background, the empty pixel, is no colour and is not counted.

A font of grey shading with one palette, a grey and a white:

```
ASCENT	6
COLORS	2
DESCENT	2
GLYPH	U+000020-000	4	0	0
GLYPH	U+000041-000	6	0	0	3AD25AE0043EDDCC6BC11E8794FCCDD3	EBBBA94934DA6897D86C5E8EA2AC5409
```

Its hash is `894ef821e147533c34b39e2131016133`.
Its A is the glyph file `3ad25ae0043eddcc6bc11e8794fccdd3.pxgl`, its palette the file `ebbba94934da6897d86c5e8ea2ac5409.hex`:

```
..0..
.010.
01.10
11111
1...1
```

```
808080
FFFFFF
```

The first pixel of the A in reading order is the grey, so it is the pixel `0`; the next colour, the white, is the pixel `1`.
The space has no glyph file, so no palette.

### KERNING

```
KERNING	U+000041-000	U+000056-000	-1
```

A kerning line has the code of the first glyph, the code of the second, and the number of pixels the second glyph moves to the right when it follows the first.
The number is an integer; a negative number moves the second glyph to the left, and a positive number has no plus sign.
A pair of adjustment 0 is not written, and a pair is written once.
The codes are those of the glyph lines, written as there.
A glyph without a code point is named by its code of the group `X`, which MUST be that of a glyph line of the file.

## Order

The lines are sorted in ascending order of their bytes, and a line that would occur twice is written once.

Since the keys are upper case and sort before the fields, the lines come key by key, in the order of the bytes of the keys: `ASCENT`, `CAPHEIGHT`, `COLORS`, `DESCENT`, `GLYPH`, `INTERNALLEADING`, `KERNING`, `LINEGAP`, `MONOSPACED`, `QUADWIDTH`, `RATIO`, `STRIKEOUTPOSITION`, `STRIKEOUTTHICKNESS`, `SYMBOLIC`, `UNDERLINEPOSITION`, `UNDERLINETHICKNESS`, `XHEIGHT`.
Within the glyph lines, as the codes all have twelve characters, they come by code, so by the letter of the group first: the internal glyphs (`I`), that is, the default glyph, then those with a code point (`U`) by code point, and last those without one (`X`) by their index.
Kerning lines come by the first code, then the second.

## Reading

A reader:

1. makes the canonical form of the file;
2. splits each line at its tabs;
3. checks every line against the rules above;
4. looks each glyph file up by its hash, in lower case, and reads it as a pxgl file, and each palette file by its hash, in lower case under `palette/`, and reads it as a hex file;
5. places each bitmap by its offsets.

A reader SHOULD check that the hash of the pxfm file and of each glyph file is its name.

A program that needs a font box, to draw or to write another format, takes `ASCENT` and `DESCENT` from the file.
When the file has none, the program may take the box from the glyphs: as many rows above the baseline as the highest bitmap reaches, and as many below it as the lowest bitmap reaches, and at least none.
That box is the program's own, and is not written into any pxfm file.

## Errors

A reader MUST reject a file:

- with a byte of 0x80 or above;
- with a line of an unknown key, too few or too many fields, or a field that is not of its form, such as an integer with a plus sign or a leading zero;
- with two lines of a key that may occur once, two lines of one kerning pair, or two glyph lines of the code `I+DEFA00-000`;
- with a negative `ASCENT` or `DESCENT`;
- with a `MONOSPACED` or `SYMBOLIC` line that has a field, or a `RATIO` that is not of the form above;
- with a `KERNING` line of adjustment 0;
- with a glyph with no inked pixels, that is, no hash, whose offsets are not both `0`;
- with a code that is not of the form `G+CCCCCC-VVV` (such as a code of version 2, `000041`, a `-` or `_DEF`), a group other than `U`, `X` and `I`, a code point above `10FFFF`, or a group `I` with another value than `DEFA00`;
- with an offset written `<x>,<y>` in one field, as version 2 of this format wrote it;
- whose codes of the group `X`, in the glyph lines, are not `X+000000-000`, `X+000001-000`, … each once, or with a `KERNING` line of a code of the group `X` that no glyph line has;
- with a glyph hash whose glyph file is not found or is not a valid pxgl file;
- with a glyph file whose colours are not numbered in reading order from `0` (see [pxgl](pxgl.md#canonical-colours));
- whose `COLORS` and palette fields break the rules of [COLORS](#colors): a `COLORS` below 2 or not the number the rules give, a glyph line with a glyph file and no palette field in a colour font, a palette field in a font with no `COLORS` line, a palette file that is not found or is not a valid [hex](hex.md) file, a palette with fewer lines than its glyph file uses colours, or a glyph file of several colours in a font with no `COLORS` line.

A file with no `ASCENT` line, no `DESCENT` line, or neither, is valid.

## Line ends in git

Git is to keep the files with LF line ends, so that a checkout on any system gives the canonical bytes.
pixelglyphlib's `.gitattributes` says so:

```
/pxfm/**/*.pxfm text eol=lf
```

## Reference implementation

- `pixelglyphlib/writers/pxfm.py`: `descriptor_lines` (the lines, the glyph files and the palette files), `descriptor_bytes`, `descriptor_hash`, `pxfm_hash`, `write`.
- `pixelglyphlib/store.py`: `font_colors` (the number of `COLORS`), `canonical_glyph`, `canonical_pixels`, `parse_palette`.
- `pixelglyphlib/readers/pxfm.py`: `read`.
- `pixelglyphlib/pxfm.py`: `store_pxfm` and `python -m pixelglyphlib.pxfm`, which writes the pxfm file of every font JSON copy under `sourcefiles/`.
