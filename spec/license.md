# license: licence text

Version 1.

A licence text file holds the text of a document that grants or limits the rights to a font: a licence, a readme with terms of use, an end-user licence agreement, or a copyright notice.
It is plain text, and it is named by the hash of its canonical form, so the same text always has the same name and is stored once, whichever fonts it belongs to.
Which font it belongs to is told by the [pxtf](pxtf.md) file that names it in a `TERMS-TEXT` line.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" are to be read as described in RFC 2119.

## Example

The CC0 1.0 Universal legal code, as the Superpowers asset packs ship it in their `LICENSE.txt`, begins:

```
CC0 1.0 Universal

Statement of Purpose

The laws of most jurisdictions throughout the world automatically confer
exclusive Copyright and Related Rights (defined below) upon the creator and
…
```

Its hash is `36ffd9dc085d529a7e60e1276d73ae5a`, so it is stored as `license/36/36ffd9dc085d529a7e60e1276d73ae5a.txt`.

## What it holds

The text of one document, whole and as it is written: not shortened, not translated, its lines not wrapped again.
A readme that holds the terms of use among other things is kept whole, since what it says around the terms can bear on them.

A document that is not plain text is stored as its text:

- a text in another encoding, such as Windows-1252 or code page 437, is decoded into Unicode;
- an HTML page, a PDF or an RTF document is its text as a reader sees it, without markup, its paragraphs separated by empty lines;
- a form feed is an empty line, and any other control character but the tab is left out.

The file names neither the document it was taken from nor where it was found.

## Encoding and canonical form

The file is UTF-8 text with no byte order mark, in Unicode Normalization Form C (NFC).
It holds no control character other than the tab and the line feed.

Its canonical form is made as follows:

1. The text is split into lines at every CR LF, lone CR and lone LF.
2. Spaces and tabs are cut from the end of each line.
3. Empty lines at the start and at the end are left out, and a run of empty lines inside is made one.
4. Each line is ended with LF, the last one too.

Tabs and spaces inside and at the start of a line are kept, so that a text laid out in columns keeps its layout.

A writer MUST write the file in canonical form.
A reader MUST make the canonical form of what it reads before it hashes it.

## Hash and file name

The hash of a file is the first 32 hex digits of the SHA-256 of its canonical form.
In a file name the hex digits are lower case, and in a pxtf file upper case.

The file is named `<hash>.txt`.
In a library it lies at `license/<first 2 digits of the hash>/<hash>.txt`.

## Errors

A reader MUST reject a file:

- that is not valid UTF-8, begins with a byte order mark, or is not in NFC;
- that holds a control character other than the tab, CR and LF;
- that is empty once made canonical.

## Line ends in git

Git is to keep the files with LF line ends:

```
/license/**/*.txt text eol=lf
```
