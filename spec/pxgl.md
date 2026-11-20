# pxgl: pixel glyph

Version 1.

A pxgl file holds the bitmap of one glyph and nothing else.
It is plain text, so it can be read and edited in any text editor, and it is named by the hash of its canonical form, so the same bitmap always has the same name.
Where the bitmap sits in the font, how far it advances the pen and which character it stands for are told by the font that uses it: by a [pxfm](pxfm.md) file, or by a font JSON of pixelglyphlib.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.

## Example

The capital letter A of a 5 by 5 pixel font:

```
..0..
.0.0.
0...0
00000
0...0
```

Its hash is `c0fc54c2ec60cb8617b041d991da1622`, so it is stored as `pxgl/c0/c0fc54c2ec60cb8617b041d991da1622.pxgl`.

## Pixels

A pixel is one of these 37 characters:

```
.0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

- `.` (U+002E) is an empty pixel: nothing is drawn there.
- `0` to `9` (U+0030 to U+0039) are the colours 0 to 9.
- `A` to `Z` (U+0041 to U+005A) are the colours 10 to 35.

The letters are upper case; a lower-case letter is not a pixel.
A pixel that is not empty is called inked.

The file does not tell what the colours are.
The font that uses the glyph does: a [pxfm](pxfm.md) file of a font of several colours names a [hex](hex.md) palette for the glyph, whose line 1 colours the pixel `0`, line 2 the pixel `1`, and so on.
A font of one colour tells no colours: its pixels are black, `0`, and the background is transparent or not specified.
A glyph of one colour therefore uses `.` and `0` only.

A glyph of more than 36 colours cannot be written in this format.

### Canonical colours

The colours of a glyph are numbered in reading order, left to right and top to bottom:

- the first inked pixel is `0`;
- the next pixel of a colour not seen yet in this glyph is `1`, the next one after that `2`, and so on through `9`, then `A` to `Z`, in the order of the 37 characters above;
- a pixel of a colour already seen has the character that colour got.

A glyph is stored in this numbering and no other, so that the same picture always gives the same file whatever numbers its source gave the colours.
A glyph whose numbering is another, such as one whose first inked pixel is `1`, or one that uses `2` before `1`, is not valid, and a reader MUST reject it.
Whoever writes a glyph with other numbers renumbers it, and renumbers the lines of its palette with it.

A glyph of `.` and `0` only is always in this numbering.

## Structure

The file is a sequence of rows, from the top row of the bitmap to the bottom one.
A row is a sequence of pixels, from the leftmost to the rightmost, ended with a line feed (LF, U+000A).

A pxgl file MUST meet all of these:

1. It is ASCII: every byte is below 0x80.
2. Once in canonical form (see below), every line consists of pixels only.
3. It has at least one row.
4. Every row has the same number of pixels, at least one.
5. At least one pixel is inked.
6. Its colours are numbered in reading order, as [above](#canonical-colours).

The width of the bitmap is the number of pixels in a row, and its height is the number of rows.
The file carries no other data: no header, no size, no comment.

## Canonical form

The canonical form of a file is made in four steps:

1. The text is split into lines at every CR LF (U+000D U+000A), lone CR (U+000D) and lone LF (U+000A).
2. Spaces (U+0020) and tabs (U+0009) are cut from the end of each line.
3. Empty lines are left out.
4. Each line is ended with LF, the last one too.

A file in canonical form is thus its rows, each ended with LF, and nothing else.
As `.` is the empty pixel, the steps never change a valid bitmap: they only undo what an editor or a transfer may have done to line ends, trailing blanks and empty lines.

A writer MUST write the canonical form.
A reader MUST make the canonical form of what it reads before it checks or reads the rows, so it accepts any of those changes.

## Hash and file name

The hash of a file is the first 32 hex digits of the SHA-256 of its canonical form, that is, the first 128 bits of the digest.
In a file name the hex digits are lower case.

The file is named `<hash>.pxgl`.
In a pixelglyphlib library it lies at `pxgl/<first 2 digits of the hash>/<hash>.pxgl`.

Since the name is the hash, two files with the same bitmap are one file, and a file whose bitmap changes gets a new name.
A reader that finds a file by its hash SHOULD check that the hash of its canonical form is its name.

A file that refers to a glyph file MAY write the hash in upper case, as a pxfm file does; it is looked up in lower case.

## Cropping

The bitmap of a glyph SHOULD be cropped to its inked pixels: its first and last row and its first and last column each have an inked pixel.
pixelglyphlib always writes a glyph cropped, and keeps where the bitmap lies in the font that uses it.

A bitmap that is not cropped is still a valid file, but it is a different file from the cropped one, with another hash.
pixelglyphlib keeps a hand-edited bitmap as it is drawn and does not crop it.

A glyph with no inked pixel, such as a space, has no pxgl file.

## Line ends in git

Git is to keep the files with LF line ends, so that a checkout on any system gives the canonical bytes.
pixelglyphlib's `.gitattributes` says so:

```
/pxgl/**/*.pxgl text eol=lf
```

## Errors

A reader MUST reject a file:

- with a byte of 0x80 or above;
- with a character, after the canonical form is made, that is not a pixel, such as a lower-case letter or a space inside a row;
- with no rows;
- whose rows differ in width;
- with no inked pixel;
- whose colours are not numbered in reading order.

## Reference implementation

- `pixelglyphlib/store.py`: `PIXELS`, `canonical`, `glyph_text`, `text_hash`, `glyph_path`, `glyph_rows`, `store_glyph`, `canonical_pixels` (renumbers the colours and the palette), `is_canonical_pixels`.
- `pixelglyphlib/update.py`: `parse` checks a hand-edited file and gives it its new name.
