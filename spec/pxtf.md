# pxtf: pixel typeface

Version 2.

A pxtf file describes a typeface: its name and its fonts.
For each font (each variant) it tells the source file the font is read from and its format, what the source states about that font, what its licence lets one do with it, how the font is read from it, and the hash of the font's [pxfm](pxfm.md) file.
Following typographic tradition, a typeface is a family of fonts, and a font is one of its variants with its own metrics and glyphs, which the pxfm file holds.
A source file can hold several fonts, such as the sizes of a Windows FON file or the 8 and the 9 dots wide reading of a VGA font dump; each is a variant, numbered from 1.
The variants of a typeface can come from one source file or from several.

The pxtf file is named by the hash of its canonical form, as pxfm and [pxgl](pxgl.md) files are.
The font JSON copies beside the source file in pixelglyphlib record that hash.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.
In the examples, the fields are separated by tab characters, and the lines of a variant begin with one.

## Example

A Windows FON file of two fonts:

```
NAME	ASI_Mono
0001
	CHARSET	0
	COPYRIGHT	(c)Copyright Altura Software Inc. 1992. All rights reserved
	PXFM	FBB120E8372CDFA484F0107DFBA46860
	SOURCE-FORMAT	FON
	SOURCE-HASH	43B49707749923470CDD7F5FCD6A552D
	SOURCE-TYPE	fnt-v2
	STYLE	Regular
0002
	CHARSET	0
	COPYRIGHT	(c) Copyright Microsoft Corp. 1987. All rights reserved.
	PXFM	7F3882B4A3788FDE3BC0111426A12C8F
	SOURCE-FORMAT	FON
	SOURCE-HASH	43B49707749923470CDD7F5FCD6A552D
	SOURCE-TYPE	fnt-v2
	STYLE	Regular
```

Its hash is `201c9ca4b9c357621f90cdf98ac47321`, so it is stored as `pxtf/20/201c9ca4b9c357621f90cdf98ac47321.pxtf`.

The typeface is named `ASI_Mono`.
Both of its fonts are read from the source file whose SHA-256 begins with `43b49707749923470cdd7f5fcd6a552d`.
It is a FON file whose fonts are read as FNT version 2 resources.
Its first font is the one whose metrics and glyphs the pxfm file `fbb120e8372cdfa484f0107dfba46860.pxfm` holds.

## What it holds

- The name of the typeface.
- For each variant: the hash of its source file, the format of that file and how it is read, what the source states about the font (its name, copyright, licence, weight name, slant, italic, underlined and struck-out, and the like), the terms of its licence as the library reads them and the hashes of the licence texts, the options that the font is read with, and the hash of its pxfm file.

Left out are the original name of the source file, where it was found, and what else the library made of the font: its id, notes, whether it is approved, and measurements such as how well its outlines lie on the pixel grid.
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

1. The top line, which tells the name of the typeface: a key and its field, separated by a tab.
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
They need not follow each other: a font of a typeface can be gone while the others keep their numbers.

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

### Words and dates

A word is one of a fixed list of values that its key names, written in lower-case ASCII letters, digits and hyphens, such as `yes` or `not-stated`.
A reader MUST reject a word that is not in the list of its key.

A date is written `YYYY-MM-DD`, as in ISO 8601, such as `2026-11-21`.

### Hashes and code points

A hash is written as 32 upper-case hex digits: the first 32 hex digits of the SHA-256 of the canonical form of the file it names.
A code point is written as six upper-case hex digits, as the `CCCCCC` of a code of the group `U` in a pxfm file, without its group and its variant.

## Top line

| Key | Fields | Count |
| --- | --- | --- |
| `NAME` | the name of the typeface, text | at most one |

`NAME` is the name of the typeface, the family name as the source states it.
A typeface whose source states no family name has no `NAME` line, unless the library gives it one.
A typeface with no variant MUST have a `NAME` line.

## Variant lines

Each key occurs at most once in a variant, except `IGNORE`, `TERMS-TEXT` and `TERMS-OBLIGATION`.
`PXFM`, `SOURCE-FORMAT`, `SOURCE-HASH` and `SOURCE-TYPE` are the keys every variant MUST have.

### PXFM

```
	PXFM	FBB120E8372CDFA484F0107DFBA46860
```

The hash of the variant's [pxfm](pxfm.md) file, in upper case.
Two variants can have the same pxfm file, when they look and measure the same.

### The source file

| Key | Fields |
| --- | --- |
| `SOURCE-FORMAT` | the format of the source file |
| `SOURCE-HASH` | the hash of the source file |
| `SOURCE-TYPE` | how the font is read from the source file |

`SOURCE-HASH` is the first 32 hex digits of the SHA-256 of the source file's bytes, in upper case.
The source file is not made canonical: its bytes are hashed as they are.

`SOURCE-FORMAT` is the format of the source file, in upper-case letters and digits.
pixelglyphlib writes `BDF`, `FON`, `OTF`, `PCF`, `PFB`, `PNG`, `RAW`, `RECORDS`, `SHEET` and `TTF`.

`SOURCE-TYPE` is a text field that names how the font is read, as pixelglyphlib knows it: `fnt-v2` and `fnt-v3` for the FNT resources of a FON file, `ttf-gid`, `otf-gid` and `pfb-gid` for outline fonts read glyph by glyph, `raw-msb` and `raw-msb-9col` for bitmap dumps, `sheet-png` for an image, and so on.

### A font made in an editor

A font can be made in a font editor instead of being read from a source file.
Its `SOURCE-FORMAT` is `PXFE` and its `SOURCE-TYPE` is `pxfe`.
Its `SOURCE-HASH` is 32 random upper-case hex digits, since there is no source file to hash.
It has no reading options: its font is its pxfm file.

A typeface can also be made in an editor; then it can have no variant, and holds its `NAME` line alone.
A variant can name a font that another typeface names too, and a variant copied from another typeface keeps its source lines.

### What the source states

Each of these keys holds a text field, as the source file states it:

| Key | What the source states |
| --- | --- |
| `COPYRIGHT` | the copyright notice |
| `DESCRIPTION` | a description of the font |
| `DESIGNER` | the name of the designer |
| `DESIGNERURL` | the web address of the designer |
| `LICENSE` | the licence |
| `LICENSEURL` | the web address of the licence |
| `NAME` | the full name of the font |
| `SLANT` | the slant, as a BDF or PCF font states it, such as `R` or `I` |
| `STYLE` | the style name, such as `Regular` or `Bold` |
| `TRADEMARK` | the trademark notice |
| `VENDOR` | the name of the vendor |
| `VENDORURL` | the web address of the vendor |
| `VERSION` | the version |
| `WEIGHTNAME` | the weight name, as a BDF or PCF font states it, such as `Medium` or `Bold` |

What the source does not state is left out.
The family name the source states is not a variant line: it is the `NAME` of the typeface.

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

### Licence terms

These keys tell what the licence of the font lets one do with it, as the library reads the terms that apply now.
Unlike the keys above, they are not copied from the source: they are the library's reading of the licence, the readme and the other terms of the font, and they are written again when that reading changes.
Every one of them can be left out; a variant with none of them has not been looked at.
They all begin with `TERMS-`, so they stand together among the lines of a variant.

| Key | Fields | Meaning |
| --- | --- | --- |
| `TERMS-CLASS` | word: licence class | the kind of licence |
| `TERMS-COMMERCIAL` | word: permission | whether the font can be used in commercial work |
| `TERMS-CONFIDENCE` | word: `high`, `medium` or `low` | how sure the reading is |
| `TERMS-DATE` | date | when the terms were last checked |
| `TERMS-DERIVED-FROM` | text | what the font is drawn after, such as `IBM VGA ROM font` |
| `TERMS-EVIDENCE` | text | the sentence of the terms that the reading rests on |
| `TERMS-LICENSE` | text | the name of the licence, such as `CC BY-SA 4.0` or `Typodermic Desktop License` |
| `TERMS-MODIFY` | word: permission | whether the font can be changed |
| `TERMS-NOTE` | text | an unusual condition, or how the terms have changed, such as `Formerly CC BY-SA, as written in this copy's font file.` |
| `TERMS-OBLIGATION` | word: obligation | a condition of the use; one line a condition |
| `TERMS-PROVENANCE` | word: provenance | where the design of the font comes from |
| `TERMS-REDISTRIBUTE` | word: permission | whether the font file can be passed on unchanged |
| `TERMS-REDISTRIBUTE-MODIFIED` | word: permission | whether a changed version can be passed on |
| `TERMS-SOURCE` | text | where the terms were read, such as `font file`, `readme` or `web research: pixelsagas.com` |
| `TERMS-SPDX` | text | the SPDX licence expression, where there is one, such as `CC0-1.0` or `OFL-1.1` |
| `TERMS-TEXT` | hash | a [licence text](license.md) of the font; one line a text |
| `TERMS-UPSTREAM-HOLDER` | text | who holds the rights of what the font is derived from |
| `TERMS-UPSTREAM-STATUS` | word: `cleared`, `not-cleared` or `unknown` | whether the rights of what the font is derived from are cleared |
| `TERMS-USE-LIMIT` | word: `personal-only` or `non-commercial` | to what use the font is limited when it is not free for commercial use |

The licence texts named in `TERMS-TEXT` lines are those of the documents that state the terms: the licence the font is under, and the readme or notice that puts it under that licence.
A text can be named although the source file does not ship it, such as the legal code of the licence that a TrueType font names in its name table.

#### Permissions

| Word | Meaning |
| --- | --- |
| `yes` | the terms allow it |
| `no` | the terms forbid it |
| `ask` | only with the permission of the holder, or with a licence bought from them |
| `not-stated` | the terms do not say, so it is not allowed without the permission of the holder |
| `unknown` | no terms of the font are known |

#### Licence classes

| Word | Licence |
| --- | --- |
| `cc0` | Creative Commons Zero |
| `public-domain` | public domain, or a licence that gives it up, such as the Unlicense |
| `ofl` | SIL Open Font License |
| `mit` | MIT licence |
| `gpl` | GNU General Public License |
| `cc-by` | Creative Commons Attribution |
| `cc-by-sa` | Creative Commons Attribution-ShareAlike |
| `cc-by-nd` | Creative Commons Attribution-NoDerivatives |
| `cc-by-nc` | Creative Commons Attribution-NonCommercial, with or without ShareAlike or NoDerivatives |
| `free` | the author's own terms, free for commercial use |
| `free-personal` | the author's own terms, free for personal use only |
| `free-unspecified` | said to be free, with no terms of what for |
| `shareware-demo` | shareware or a demo version |
| `proprietary` | the holder's, with no licence granted |
| `unknown` | no licence found |

#### Obligations

| Word | Meaning |
| --- | --- |
| `attribution` | the author must be credited |
| `device-limit` | the licence covers a limited number of computers |
| `embed-only` | the font can be passed on only embedded in a work, not as a font file |
| `include-license` | the licence text must go with the font |
| `include-notice` | the copyright notice or the readme must go with the font |
| `no-collections` | the font cannot be put into a font collection or archive |
| `no-embedding` | the font cannot be embedded in software or documents |
| `no-nft-crypto` | the font cannot be used for NFTs or cryptocurrencies |
| `no-sale` | the font cannot be sold |
| `no-sale-alone` | the font cannot be sold by itself, only as part of a larger work |
| `notify-author` | the author must be told of the use |
| `only-bundled-in-project` | the font can be passed on only as part of one's own project |
| `rename` | a changed version must have another name |
| `rename-if-reserved-name` | a changed version cannot have the Reserved Font Name of the licence |
| `share-alike` | a changed version must be under the same licence |

#### Provenance

| Word | Meaning |
| --- | --- |
| `original` | the author's own design |
| `vendor-original` | the design of a vendor, such as the font of a BIOS or a video ROM |
| `derived` | drawn after another font or lettering, which `TERMS-DERIVED-FROM` names |
| `possibly-derived` | maybe drawn after another one, which `TERMS-DERIVED-FROM` names |
| `unknown` | not known |

`TERMS-DERIVED-FROM`, `TERMS-UPSTREAM-HOLDER` and `TERMS-UPSTREAM-STATUS` are written only with `derived` or `possibly-derived`.
`TERMS-USE-LIMIT` is written only when `TERMS-COMMERCIAL` is not `yes`.

#### Showing the terms

A program that shows a font, such as a font editor, can show its terms as a table of the permissions in plain words, with the conditions below them and the licence texts a click away:

| | Generic Mobile System Nuevo Regular |
| --- | --- |
| Licence | CC0 1.0 (`CC0-1.0`) |
| Commercial use | allowed |
| Changing it | allowed |
| Passing it on | allowed |
| Passing on a changed version | allowed |
| Conditions | none |
| Design | the author's own |
| Read from | font file, high confidence, 2026-11-21 |
| Licence texts | `36FFD9DC085D529A7E60E1276D73AE5A` |

The words are shown by the meanings in the tables above.

### Reading options

These say how the variant is read from the source file, as far as the variant number and `SOURCE-TYPE` do not:

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

The top line comes first.
The variants follow in ascending order of their numbers, each number followed by the lines of that variant, sorted by their bytes.
A line that would occur twice in one place is written once.

The lines of a variant thus come in the alphabetical order of their keys, since the keys are upper case, a hyphen sorts before any letter, and a tab sorts before a hyphen, so a key comes before the longer keys that begin with it: `LICENSE` before `LICENSEURL`, and `TERMS-REDISTRIBUTE` before `TERMS-REDISTRIBUTE-MODIFIED`.
The `TERMS-` lines come last but for `UNDERLINED`, `VENDOR`, `VENDORURL`, `VERSION` and `WEIGHTNAME`.
The `SOURCE-` lines come in the order `SOURCE-FORMAT`, `SOURCE-HASH`, `SOURCE-TYPE`, after `SLANT` and before `STYLE`.
The lines of the three keys without a field are the bare keys, which sort as the keys do.
Two `IGNORE` lines come in the order of their first code points, two `TERMS-TEXT` lines in the order of their hashes, and two `TERMS-OBLIGATION` lines in the alphabetical order of their words.

A reader MUST reject a file whose lines are not in this order.

## Errors

A reader MUST reject a file:

- that is not valid UTF-8, begins with a byte order mark, or is not in NFC;
- with a line of an unknown key, too few or too many fields, or a field that is not of its form;
- with a text field that has a backslash that is no escape;
- with a top line of a key other than `NAME`, or with two `NAME` top lines;
- with no variant and no `NAME` line, or with two variants of one number;
- with a top line after a variant number, or a variant line before any;
- with a variant that has no `PXFM`, `SOURCE-FORMAT`, `SOURCE-HASH` or `SOURCE-TYPE` line, or two lines of a key other than `IGNORE`, `TERMS-TEXT` and `TERMS-OBLIGATION`;
- with a variant that has `TERMS-DERIVED-FROM`, `TERMS-UPSTREAM-HOLDER` or `TERMS-UPSTREAM-STATUS` and no `TERMS-PROVENANCE` of `derived` or `possibly-derived`, or `TERMS-USE-LIMIT` with `TERMS-COMMERCIAL` `yes`;
- whose lines are not in canonical order.

## Changes from version 1

- `FORMAT`, `HASH` and `TYPE` were top lines; they are now the variant lines `SOURCE-FORMAT`, `SOURCE-HASH` and `SOURCE-TYPE`, which every variant has, so the variants of a typeface can come from several source files.
- The top line `NAME` is new: the name of the typeface.
- The variant line `FAMILY` is gone; the family name is the `NAME` of the typeface.
- The variant line `FULLNAME` is now `NAME`.
- The licence terms are new: the `TERMS-` keys, and the [licence text](license.md) files that `TERMS-TEXT` names.

## Line ends in git

Git is to keep the files with LF line ends:

```
/pxtf/**/*.pxtf text eol=lf
```

## Reference implementation

- `pixelglyphlib/pxtf.py`: `typeface_lines`, `typeface_bytes`, `pxtf_hash` and `store_pxtf` write the file, `read` reads and checks it.
- `python -m pixelglyphlib.pxtf` writes the pxtf file of every source file that has font JSON copies.
