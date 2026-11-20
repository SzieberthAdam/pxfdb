# pxtf: pixel typeface

Version 1.

A pxtf file describes a typeface: the fonts of one source file.
It tells the format of the source file, and for each font read from it (each variant) what the source states about that font, how the font is read from it, and the hash of the font's [pxfm](pxfm.md) file.
Following typographic tradition, the typeface is the family as the source file gives it, and a font is one of its variants with its own metrics and glyphs, which the pxfm file holds.
A source file can hold several fonts, such as the sizes of a Windows FON file or the 8 and the 9 dots wide reading of a VGA font dump; each is a variant, numbered from 1.

The pxtf file is named by the hash of its canonical form, as pxfm and [pxgl](pxgl.md) files are.
The font JSON copies beside the source file in pixelglyphlib record that hash.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.
In the examples, the fields are separated by tab characters, and the lines of a variant begin with one.

## Example

A Windows FON file of two fonts:

```
FORMAT	FON
HASH	43B49707749923470CDD7F5FCD6A552D
TYPE	fnt-v2
0001
	CHARSET	0
	COPYRIGHT	(c)Copyright Altura Software Inc. 1992. All rights reserved
	FAMILY	ASI_Mono
	PXFM	FBB120E8372CDFA484F0107DFBA46860
	STYLE	Regular
0002
	CHARSET	0
	COPYRIGHT	(c) Copyright Microsoft Corp. 1987. All rights reserved.
	FAMILY	ASI_Mono
	PXFM	7F3882B4A3788FDE3BC0111426A12C8F
	STYLE	Regular
```

Its hash is `97d45d22be0ae656b9e567288f7dbe7b`, so it is stored as `pxtf/97/97d45d22be0ae656b9e567288f7dbe7b.pxtf`.

The source file is the one whose SHA-256 begins with `43b49707749923470cdd7f5fcd6a552d`.
It is a FON file whose fonts are read as FNT version 2 resources.
Its first font is the one whose metrics and glyphs the pxfm file `fbb120e8372cdfa484f0107dfba46860.pxfm` holds.

## What it holds

- The hash of the source file, its format and how it is read.
- For each variant: what the source states about the font (names, copyright, licence, weight name, slant, italic, underlined and struck-out, and the like), the options that the font is read with, and the hash of its pxfm file.

Left out are the original name of the source file, where it was found, and what the library made of the font: its id, notes, whether it is approved, and measurements such as how well its outlines lie on the pixel grid.
What the font looks like and measures is in its pxfm file, not here.

## Encoding and canonical form

The file is UTF-8 text with no byte order mark.
Every text value is in Unicode Normalization Form C (NFC).

Its canonical form is made as that of a pxfm file, but in UTF-8:

1. The text is split into lines at every CR LF, lone CR and lone LF.
2. Spaces and tabs are cut from the end of each line.
3. Empty lines are left out.
4. Each line is ended with LF, the last one too.

A writer MUST write the file in canonical form, with its lines in canonical order (see [Order](#order)).
A reader MUST make the canonical form of what it reads before it reads the lines.

## Hash and file name

The hash of a file is the first 32 hex digits of the SHA-256 of its canonical form.
In a file name the hex digits are lower case.

The file is named `<hash>.pxtf`.
In a pixelglyphlib library it lies at `pxtf/<first 2 digits of the hash>/<hash>.pxtf`, and each font JSON copy of the source under `sourcefiles/` names it in its `pxtf` field.

A source file has one pxtf file at a time: when a font of it is added, changed or removed, its pxtf file is written again under a new hash, and the old one stays until it is deleted by hand with `python -m pixelglyphlib.remove pxtf <hash>`.

## Structure

The file has three parts, in this order:

1. The top lines, which tell about the source file: a key and its fields, separated by tabs.
2. For each variant, in ascending order, a line of its number alone.
3. After each variant number, the lines of that variant: a tab, then a key and its fields, separated by tabs.

```
KEY	FIELD
0001
	KEY	FIELD	FIELD…
```

No field is empty.
A field holds no tab and no line end; in a text field these are escaped (see [Text](#text)), and the other fields are of forms that have none.

### Variant numbers

A variant number is written in decimal with at least four digits, padded with zeros: `0001`, `0002`, `0010`, `12345`.
Variant numbers begin at 1, and no number occurs twice.
They need not follow each other: a font of a source can be gone while the others keep their numbers.

### Text

A text field holds any Unicode text, in NFC, with these escapes, so that it stays on one line:

| Character | Written |
| --- | --- |
| backslash (U+005C) | `\\` |
| tab (U+0009) | `\t` |
| line end: LF, CR LF or CR | `\n` |
| any other control character (U+0000 to U+001F, U+007F) | `\x` and two upper-case hex digits, such as `\x01` |

A backslash is followed by one of `\`, `t`, `n`, or `x` and two upper-case hex digits; a reader MUST reject any other.
Spaces at the end of a value are left out, as the canonical form would cut them.
A value that is empty once escaped and cut is left out with its line.

### Numbers

An integer is written in decimal, with a minus sign if it is negative, no plus sign and no leading zeros.
A number that can be a fraction is written as an integer if it is a whole number: `125`, not `125.0`.
Otherwise it is written as the shortest decimal that reads back as the same IEEE 754 double-precision number, with a decimal point (`.`) and at least one digit on each side of it, such as `44.487` or `0.30000000000000004`.
If its decimal exponent is below -4 or 16 and above, it is written in scientific notation with an upper-case `E` and a signed exponent of at least two digits, such as `1E-05`.
This is Python's `repr` of a `float`, made upper case.

### Hashes and code points

A hash is written as 32 upper-case hex digits: the first 32 hex digits of the SHA-256 of the file it names, or of its canonical form for a pxfm file.
A code point is written as six upper-case hex digits, as the `CCCCCC` of a code of the group `U` in a pxfm file, without its group and its variant.

## Top lines

| Key | Fields | Count |
| --- | --- | --- |
| `FORMAT` | the format of the source file | exactly one, here or in every variant |
| `HASH` | the hash of the source file | exactly one |
| `TYPE` | how the fonts are read from the source file | exactly one, here or in every variant |

`HASH` is the first 32 hex digits of the SHA-256 of the source file's bytes, in upper case.
The source file is not made canonical: its bytes are hashed as they are.

`FORMAT` is the format of the source file, in upper-case letters and digits.
pixelglyphlib writes `BDF`, `FON`, `OTF`, `PCF`, `PFB`, `RAW`, `RECORDS`, `SHEET` and `TTF`.

`TYPE` is a text field that names how the fonts are read, as pixelglyphlib knows it: `fnt-v2` and `fnt-v3` for the FNT resources of a FON file, `ttf-gid`, `otf-gid` and `pfb-gid` for outline fonts read glyph by glyph, `raw-msb` and `raw-msb-9col` for bitmap dumps, `sheet-png` for an image, and so on.

When all the variants have the same `FORMAT`, it is a top line; when they differ, every variant has its own `FORMAT` line and there is no top line.
The same goes for `TYPE`.
A reader MUST reject a file that has a `FORMAT` or `TYPE` both at the top and in a variant, has it neither at the top nor in every variant, or has the same one in every variant.

## Variant lines

Each key occurs at most once in a variant, except `IGNORE`.
`PXFM` is the only key every variant MUST have.

### PXFM

```
	PXFM	FBB120E8372CDFA484F0107DFBA46860
```

The hash of the variant's [pxfm](pxfm.md) file, in upper case.
Two variants can have the same pxfm file, when they look and measure the same.

### What the source states

Each of these keys holds a text field, as the source file states it:

| Key | What the source states |
| --- | --- |
| `COPYRIGHT` | the copyright notice |
| `DESCRIPTION` | a description of the font |
| `DESIGNER` | the name of the designer |
| `DESIGNERURL` | the web address of the designer |
| `FAMILY` | the family name |
| `FULLNAME` | the full name |
| `LICENSE` | the licence |
| `LICENSEURL` | the web address of the licence |
| `SLANT` | the slant, as a BDF or PCF font states it, such as `R` or `I` |
| `STYLE` | the style name, such as `Regular` or `Bold` |
| `TRADEMARK` | the trademark notice |
| `VENDOR` | the name of the vendor |
| `VENDORURL` | the web address of the vendor |
| `VERSION` | the version |
| `WEIGHTNAME` | the weight name, as a BDF or PCF font states it, such as `Medium` or `Bold` |

What the source does not state is left out.

Three more keys have no field at all, a key alone on its line with no tab after it.
Each is written only when the source says that this variant has the property:

| Key | What the source states |
| --- | --- |
| `ITALIC` | the font is italic |
| `STRUCKOUT` | the font is a struck-out variant, as the flag of a FON file says |
| `UNDERLINED` | the font is an underlined variant, as the flag of a FON file says |

They are variant properties of the source and not metrics of the glyphs, so they are here and not in the [pxfm](pxfm.md) file.
A reader MUST reject one of them with a field.

For a source that states nothing, such as a raw dump, these are what the library's descriptor of it says.

### Reading options

These say how the variant is read from the source file, as far as the variant number and `TYPE` do not:

| Key | Fields | Meaning |
| --- | --- | --- |
| `ADVANCEGAP` | integer | records: what is added to the width of a glyph to give its advance |
| `CELL` | `<width>x<height>` | raw, sheet and records: the size of a cell in pixels, with a lower-case `x` |
| `CHARSET` | integer | FON: the character set the FNT resource states, which tells how its codes map to code points |
| `CODEPAGE` | text | raw, sheet and records: the code page that maps the cells to code points, such as `cp437-graphic` |
| `GRID` | x, y, x phase, y phase, and `PERGLYPH` or nothing | outline fonts: the pixel grid in font units, see below |
| `IGNORE` | first and last code point | outline fonts: a range of code points left out, as their glyphs are no pixel glyphs; one line a range |
| `NINTHCOLUMN` | first and last code, integers | raw, 8 pixels wide cells: the codes whose ninth column copies the eighth, as a VGA in 9-dot mode does |
| `PFM` | hash | PFB: the hash of the PFM file read with it |
| `RECORDSIZE` | integer | records: the size of a record in bytes |
| `SPACEADVANCE` | integer | records: the advance of the space, where it is not its width plus `ADVANCEGAP` |

`GRID` gives the width and the height of a pixel in font units, and where the cells start along x and y, as numbers that can be fractions.
`PERGLYPH` at the end says that each glyph has its own phases, found glyph by glyph.
A font read with square pixels of one size from the origin is `GRID	125	125	0	0`.

```
	GRID	44.487	44.487	0	0	PERGLYPH
	IGNORE	0000C0	0000FF
```

A reader MUST reject a variant line of any other key.

## Order

The top lines come first, sorted by their bytes.
The variants follow in ascending order of their numbers, each number followed by the lines of that variant, sorted by their bytes.
A line that would occur twice in one place is written once.

The top lines thus come in the order `FORMAT`, `HASH`, `TYPE`, and the lines of a variant in the alphabetical order of their keys, since the keys are upper case and a tab sorts before any letter.
The lines of the three keys without a field are the bare keys, which sort as the keys do.
Two `IGNORE` lines come in the order of their first code points.

A reader MUST reject a file whose lines are not in this order.

## Errors

A reader MUST reject a file:

- that is not valid UTF-8, begins with a byte order mark, or is not in NFC;
- with a line of an unknown key, too few or too many fields, or a field that is not of its form;
- with a text field that has a backslash that is no escape;
- with no `HASH` line, or with two top lines of one key;
- with no variant, or with two variants of one number;
- with a top line after a variant number, or a variant line before any;
- with a variant that has no `PXFM` line, or two lines of a key other than `IGNORE`;
- whose `FORMAT` or `TYPE` lines break the rules of [Top lines](#top-lines);
- whose lines are not in canonical order.

## Line ends in git

Git is to keep the files with LF line ends:

```
/pxtf/**/*.pxtf text eol=lf
```

## Reference implementation

- `pixelglyphlib/pxtf.py`: `typeface_lines`, `typeface_bytes`, `pxtf_hash` and `store_pxtf` write the file, `read` reads and checks it.
- `python -m pixelglyphlib.pxtf` writes the pxtf file of every source file that has font JSON copies.
