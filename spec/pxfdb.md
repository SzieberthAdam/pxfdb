# pxfdb: pixel font database

Version 1.

A pxfdb file is an SQLite database that holds the whole text database of a pixelglyphlib library: every [pxgl](pxgl.md) glyph file, every [hex](hex.md) palette file, every [pxfm](pxfm.md) metrics file and every [pxtf](pxtf.md) typeface file, orphans included.
It is made for software and websites that have to query the library efficiently: which fonts hold a glyph, how alike two fonts are, which glyphs are drawn for a code point, which fonts have a given height.

The text files stay the master and the version-controlled form.
The two forms convert into each other losslessly: a database built from the folders gives back every file of them byte for byte, and a database exported into an empty folder gives the folders again.
The database is a generated file; it is not in git, and it is built again whenever the files change.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.

## Files and commands

The default file is `pxfdb.pxfdb` in the root of the library (it is in `.gitignore`).
The extension is `.pxfdb`; the content is a plain SQLite 3 database.

```bash
python -m pixelglyphlib.pxfdb build [--db FILE] [--root DIR]
python -m pixelglyphlib.pxfdb export [--db FILE] [--root DIR]
```

The reference implementation is `pixelglyphlib/pxfdb.py`; its query subcommands are only examples of how the database can be queried (see the README).

## Keys and types

- A file is named by its hash, and the hash is its primary key and its reference: 16 bytes in a `BLOB`, the first 128 bits of the SHA-256 that gives the first 32 hex digits of the name.
  The case of the digits does not matter in a query, as `x'C0FC54C2…'` and `x'c0fc54c2…'` are the same blob; software converts a hex string with `unhex` (SQLite 3.41 and later) or in its own language.
- A code of a pxfm file, `G+CCCCCC-VVV`, is the integer of [pxfm](pxfm.md#the-code-as-a-number): the ten hex digits `GCCCCCCVVV`, the group `0` for `U`, `A` for `X` and `F` for `I`.
  `U+000041-000` is 0x0000041000, the first symbol `X+000000-000` is 0xA000000000 and the default glyph `I+DEFA00-000` is 0xFDEFA00000.
  The codes of the code points are those below 0x1000000000, and the code point of such a code is the code shifted right by 12 bits (see [pxfm_glyph](#pxfm_glyph)).
- A colour of a palette is four bytes, `RR`, `GG`, `BB`, `AA`, with `AA` = `FF` when the file wrote six digits (see [palette](#palette)).
- A table of which a row is addressed by its primary key is `WITHOUT ROWID`, so the key is the row's place in the file and costs no second copy.
- Foreign keys are declared but not enforced by SQLite (`PRAGMA foreign_keys` is off by default); `build` checks them all with `PRAGMA foreign_key_check` before it finishes.

## Schema

The format and its version are in `meta`; a program SHOULD refuse a file with another format or version.

```sql
CREATE TABLE meta (
    key   TEXT PRIMARY KEY,
    value TEXT NOT NULL
) WITHOUT ROWID;
```

`meta` holds two rows: `format` = `PXFDB` and `format_version` = `1`.

### glyph

```sql
CREATE TABLE glyph (
    hash   BLOB PRIMARY KEY,
    width  INTEGER NOT NULL,
    height INTEGER NOT NULL,
    pixels TEXT NOT NULL,
    CHECK (length(hash) = 16 AND width > 0 AND height > 0 AND length(pixels) = width * height)
) WITHOUT ROWID;
```

One row per pxgl file.
`pixels` is the rows of the file concatenated, top row first, each pixel one of `.0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ`; it has `width * height` characters.
The file is `pixels` cut into rows of `width` characters, each ended with LF, so there is no copy of the text and the file comes back byte for byte.

### pxfm

```sql
CREATE TABLE pxfm (
    hash                BLOB PRIMARY KEY,
    ascent              INTEGER CHECK (ascent >= 0),
    descent             INTEGER CHECK (descent >= 0),
    line_gap            INTEGER,
    cap_height          INTEGER,
    x_height            INTEGER,
    internal_leading    INTEGER,
    quad_width          INTEGER,
    ratio_x             INTEGER CHECK (ratio_x > 0),
    ratio_y             INTEGER CHECK (ratio_y > 0),
    underline_position  INTEGER,
    underline_thickness INTEGER,
    strikeout_position  INTEGER,
    strikeout_thickness INTEGER,
    monospaced          INTEGER NOT NULL CHECK (monospaced IN (0, 1)),
    symbolic            INTEGER NOT NULL CHECK (symbolic IN (0, 1)),
    colors              INTEGER CHECK (colors > 1),
    height              INTEGER,
    CHECK (length(hash) = 16),
    CHECK (height IS ascent + descent),
    CHECK ((ratio_x IS NULL) = (ratio_y IS NULL))
) WITHOUT ROWID;
```

One row per pxfm file, with what the file states about the font as a whole.

| Column | The line of the file |
| --- | --- |
| `ascent`, `descent`, `line_gap`, `cap_height`, `x_height`, `internal_leading`, `quad_width` | `ASCENT`, `DESCENT`, `LINEGAP`, `CAPHEIGHT`, `XHEIGHT`, `INTERNALLEADING`, `QUADWIDTH`; `NULL` when the file has no such line |
| `underline_position`, `underline_thickness`, `strikeout_position`, `strikeout_thickness` | `UNDERLINEPOSITION`, `UNDERLINETHICKNESS`, `STRIKEOUTPOSITION`, `STRIKEOUTTHICKNESS`; `NULL` when the file has none |
| `ratio_x`, `ratio_y` | `RATIO` `A:B` as `A` and `B`; both `NULL` when the file has none |
| `monospaced` | 1 when the file has a `MONOSPACED` line, else 0 |
| `symbolic` | 1 when the file has a `SYMBOLIC` line, else 0 |
| `colors` | the number of the `COLORS` line; `NULL` when the file has none, as a font of one colour has not |
| `height` | `ascent + descent`, `NULL` when either is `NULL`; derived, and kept equal to it by a `CHECK` |

### palette

```sql
CREATE TABLE palette (
    hash     BLOB PRIMARY KEY,
    colors   BLOB NOT NULL,
    colorset BLOB NOT NULL,
    CHECK (length(hash) = 16 AND length(colors) > 0 AND length(colors) % 4 = 0 AND length(colorset) % 4 = 0)
) WITHOUT ROWID;
```

One row per hex file.
`colors` is the palette in the order of the file, as raw RGBA, four bytes a colour: line 1 first.
The alpha is `FF` where the line has six digits, so the file comes back with six digits for a colour of alpha `FF` and eight otherwise, as the specification demands.
`colorset` is the same colours sorted by their four bytes, with no duplicate (a palette has none), the canonical set of the palette: two palettes of the same colours in another order have the same `colorset`, so palettes can be searched and compared by it.

### pxfm_glyph

```sql
CREATE TABLE pxfm_glyph (
    pxfm    BLOB NOT NULL REFERENCES pxfm,
    code    INTEGER NOT NULL CHECK (code >= 0),
    x       INTEGER NOT NULL,
    y       INTEGER NOT NULL,
    advance INTEGER NOT NULL,
    glyph   BLOB REFERENCES glyph,
    palette BLOB REFERENCES palette,
    PRIMARY KEY (pxfm, code)
) WITHOUT ROWID;
```

One row per `GLYPH` line.
`code` is the code of the line as an integer (see [Keys and types](#keys-and-types)); `x` and `y` are the offsets; `glyph` is the hash of the pxgl file, `NULL` for a glyph with no pixels; `palette` is the hash of the glyph's hex file, `NULL` in a font that is not a colour font (that has no `COLORS` line) and for a glyph with no pixels.
The default glyph is the row with the code 0xFDEFA00000, the largest code.
A code from 0x1000000000 on is no code point: a symbol of the group `X`, whose code only names it within its font, or an internal glyph of the group `I`.
The variants of a glyph lie in the codes after it, so the glyph of the code point U+0041 and its variants are the rows from 0x0000041000 to 0x0000041FFF.
A pxfm file with two glyph lines of one code is not in the specification, and `build` rejects it.
The key `(pxfm, code)` keeps the lines of a font together, so reading them, or their glyph set, is a range of the table.

### pxfm_kerning

```sql
CREATE TABLE pxfm_kerning (
    pxfm   BLOB NOT NULL REFERENCES pxfm,
    first  INTEGER NOT NULL,
    second INTEGER NOT NULL,
    adjust INTEGER NOT NULL CHECK (adjust <> 0),
    PRIMARY KEY (pxfm, first, second)
) WITHOUT ROWID;
```

One row per `KERNING` line, with the codes as integers, as in [pxfm_glyph](#pxfm_glyph).

### pxtf

```sql
CREATE TABLE pxtf (
    hash   BLOB PRIMARY KEY,
    source BLOB NOT NULL,
    CHECK (length(hash) = 16 AND length(source) = 16)
) WITHOUT ROWID;
```

One row per pxtf file; `source` is its `HASH` line, the hash of the source file.

### pxtf_variant

```sql
CREATE TABLE pxtf_variant (
    pxtf   BLOB NOT NULL REFERENCES pxtf,
    number INTEGER NOT NULL CHECK (number > 0),
    pxfm   BLOB NOT NULL REFERENCES pxfm,
    format TEXT NOT NULL,
    type   TEXT NOT NULL,
    PRIMARY KEY (pxtf, number)
) WITHOUT ROWID;
```

One row per variant.
`number` is the variant number as an integer; it is written with at least four digits.
`pxfm` is its `PXFM` line.
`format` and `type` are those of the file's top lines, or of the variant's own lines when the variants differ; the text is unescaped.

### pxtf_field

```sql
CREATE TABLE pxtf_field (
    pxtf   BLOB NOT NULL,
    number INTEGER NOT NULL,
    key    TEXT NOT NULL,
    value  TEXT NOT NULL,
    PRIMARY KEY (pxtf, number, key, value),
    FOREIGN KEY (pxtf, number) REFERENCES pxtf_variant
) WITHOUT ROWID;
```

One row per other line of a variant, so a repeated key (`IGNORE`) is several rows and a key can be added to the format without a change of the schema.
`key` is the upper-case key.
`value` is, for a key with a text field (`COPYRIGHT`, `DESCRIPTION`, `DESIGNER`, `DESIGNERURL`, `FAMILY`, `FULLNAME`, `LICENSE`, `LICENSEURL`, `SLANT`, `STYLE`, `TRADEMARK`, `VENDOR`, `VENDORURL`, `VERSION`, `WEIGHTNAME`, `CODEPAGE`), the text with its escapes undone; for a key without a field (`ITALIC`, `STRUCKOUT`, `UNDERLINED`), the empty string; for any other key, its fields joined by tabs, as written (`GRID` `44.487\t44.487\t0\t0\tPERGLYPH`, `IGNORE` `0000C0\t0000FF`, `CELL` `8x16`, `CHARSET` `0`, `PFM` the 32 digits in upper case).

## Indexes

Every index is made after the data is loaded, and `ANALYZE` is run so that the query planner knows the sizes.
Every table is `WITHOUT ROWID`, so its primary key is its clustered index; the indexes listed here are the others.

```sql
CREATE INDEX glyph_size ON glyph (width, height);
CREATE INDEX palette_by_colorset ON palette (colorset);
CREATE INDEX pxfm_glyph_by_glyph ON pxfm_glyph (glyph, pxfm) WHERE glyph IS NOT NULL;
CREATE INDEX pxfm_glyph_by_code ON pxfm_glyph (code, glyph, pxfm) WHERE code < 68719476736;
```

| Index | For |
| --- | --- |
| `glyph_size` | glyphs by size |
| `palette_by_colorset` | the palettes of the same colours, in whatever order |
| `pxfm_glyph_by_glyph` | the fonts that hold a glyph |
| `pxfm_glyph_by_code` | the glyphs of a code point and their variants; the rows of codes from 0x1000000000 (68719476736) on are not in it, so a query adds `AND code < 68719476736` for SQLite to use it |

These are the lookups that would scan `pxfm_glyph` (about 300,000 rows) or `glyph` (about 155,000).
The other tables are small (about 1,170 `pxfm` and 1,040 `pxtf` rows) and have no index but their primary keys; a query on them scans them.

## Converting the files into the database

`build` reads every file under `pxgl/`, `palette/`, `pxfm/` and `pxtf/` of the root, and checks it before it is stored.
It stops at the first file that fails, and the old database, if there is one, stays as it is.

1. Each file lies at `<folder>/<2 hex digits>/<hash>.<extension>` in lower case (the extension of the folder `palette/` is `hex`), any other name or place is an error; it is in canonical form already, and the hash of its canonical form is its name.
2. A pxgl file is read as [pxgl](pxgl.md) says: rows of equal width, only pixels, at least one inked, the colours numbered in reading order.
   The row of `glyph` is made of its width, its height and the concatenated rows.
3. A hex file is read as [hex](hex.md) says: valid colours in upper case, an alpha of `FF` not written, no colour twice, no empty line.
   The row of `palette` is made of its colours as RGBA bytes in the order of the file and the same sorted.
4. A pxfm file is read as [pxfm](pxfm.md) says, with the rules of its reader: keys and fields, integers, the codes `G+CCCCCC-VVV`, hashes, `RATIO`, kerning, the consecutive codes of the group `X` and the default glyph's line `I+DEFA00-000`, the `COLORS` line and the palette fields against the glyph files and the palettes (the number of colours, a palette of enough lines, a palette field in a colour font only), and that every glyph file and every palette file it names is in the library.
   Its lines go to the columns and tables above.
   The lines must be the lines the writer would write: sorted by their bytes, each once; a file that is not is an error.
5. A pxtf file is read by the strict reader of [pxtf](pxtf.md), and must be what the writer would write from what was read.
   A pxfm file it names must be in the library.
   The top lines `FORMAT` and `TYPE` are copied into every variant; they are put at the top again on export when all the variants are the same, as the specification demands.
6. The glyph files, the palette files and the pxfm files that nothing names are stored all the same.

## Converting the database into the files

`export` writes `pxgl/<2>/<hash>.pxgl`, `palette/<2>/<hash>.hex`, `pxfm/…` and `pxtf/…` under the root, from the tables.

- A pxgl file is the rows of `pixels`, each ended with LF.
  A glyph whose colours are not numbered in reading order is refused.
- A hex file is the colours of `colors`, a line each, six upper-case hex digits when the alpha is `FF` and eight otherwise, each ended with LF.
- A pxfm file is made of the lines of its columns and rows, in the form of the specification: the metric lines `KEY<TAB>integer` of the columns that are not `NULL`, `RATIO<TAB>A:B`, `MONOSPACED`, `SYMBOLIC`, `COLORS<TAB>colors` when `colors` is not `NULL`, `GLYPH<TAB>code<TAB>advance<TAB>x<TAB>y[<TAB>HASH[<TAB>PALETTE]]` from the rows of `pxfm_glyph`, the code written `G+CCCCCC-VVV` and the hashes in upper case, with none for a `NULL` glyph and none for a `NULL` palette, and `KERNING<TAB>first<TAB>second<TAB>adjust`; then the lines are sorted by their bytes and each ended with LF.
- A pxtf file has the `HASH` line, `FORMAT` and `TYPE` at the top when all the variants have the same, else in each variant; then each variant as its number with at least four digits and its lines: `PXFM` and the rows of `pxtf_field` (text values escaped as in [pxtf](pxtf.md#text), keys without a field alone), sorted by their bytes, each after a tab.

The hash of each file made is checked against its key before anything is written.
Then a first pass looks at every target path: a file that is there with the same bytes is left, and if one is there with other bytes (or a folder is in the way), `export` stops with an error before it has written a file.
A second pass writes the files that are not there.
`export` never deletes or overwrites a file, so it can be run into the library itself to restore what is missing.
