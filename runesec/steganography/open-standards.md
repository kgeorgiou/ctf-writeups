**Name**: Open Standards  
**Points**: 60  
**Instructions**: Eeeehh, What's up doc? open-standards.docx 
**Hint**: XML is awesome! /sarcasm

## Exploration

- We are given a file named `open-standards.docx`

- Let's check its file type:
```
$ file open-standards.docx
open-standards.docx: Microsoft OOXML
```

- Looking up "Microsoft OOXML" should give us some info on how to open and read this kind of files.

## Exploitation

Before looking up anything online, let's try reading the contents of this file with good ol' vim.

```
$ vim open-standards.docx
" zip.vim version v27
" Browsing zipfile open-standards.docx
" Select a file with cursor and press ENTER

[Content_Types].xml
docProps/
docProps/app.xml
docProps/core.xml
_rels/
_rels/.rels
word/
word/settings.xml
word/document.xml
word/fontTable.xml
word/styles.xml
word/_rels/
word/_rels/document.xml.rels
word/.settings.xml.swp
```

Splendid. Vim is able to unzip the file and lets us select and read the contents of each file within.  

The flag is sitting at the end of the last file in the list, `word/.settings.xml.swp`.