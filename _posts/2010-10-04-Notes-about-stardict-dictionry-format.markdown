---
layout: post
title:  "Notes about Stardict dictionary format"
date:   2010-10-04 22:48:45
description: 
categories:
- blog
---

This text covers some parts of Stardict format as it were at version 2.4.2

## Uncompressed Stardict dictionary

Uncompressed Stardict dictionary consists of three files: .ifo, .dic and .idx. These files should have the same base name, and should be stored in stardict
dictionary dir (i.e. /usr/share/stardict/dic), or in any subdir inside that dir. Stardict automatically creates .idx.oft file for internal use.
This file is automatically regenerated when stardict finds new .idx file

### .dic file 
For uncompressed dictionary .dic file is a plaintext utf-8 file in witch dictionary articles are stored one by one without any reasonable separation.
Information about how to separate one article from another is stored in .idx file

### .idx file 
Stardict .idx file is a binary file filled with records with following structure:

<pre>[Word: n bytes][0: 1 byte][Article-start: 4 bytes][Article-length: 4 bytes]</pre>

Word             - is the word article is indexed on, this field has variable length, the string is zero-terminated, so the next bye is always:
0                    - the marker of the end of Word field
Article-start    - Location of the start of article in .dic file in bytes.
Article-length   - How many bytes should be read form the beginning of the article to get in whole.

Be sure that words in .idx file are: 1. sorted; 2. sorted in posix locale. Information from .idx file is directly used for 
tree-search (Chek if this word is right), using strcmp C function. If your words are not ordered in proper way, some articles can 
be simply not found.

### .ifo file

Stardict .ifo file is used for soring dictionary meta-information. It is plain-text files with option=value lines in it:

<pre>StarDict's dict ifo file
version=[Version: this spesification discribes 2.4.2]
wordcount=[Number of words in dictionary]
bookname=[Name of the dictionary]
idxfilesize=[Size of .idx file in bytes]
date=[Date in YYYY-MM-DD, the date whe dictioany were created]
sametypesequence=[type of the dictionary, one letter. Put 'x' for uncimpressed dictionary]</pre>


Be sure to specify real values for wordcount and idxfilesize. These values would be used for allocating memory and scanning of the index file. 
If these values are wrong, Statdict will not work properly.

bookname and date options are for display purposes only, and can be assigned in any way


## Compressed Stardict dictionay

This articles does not covers compressed dictionary format for now. About this dictionary I can tell two things for sure:

1. Instead on .dic, compressed Stardict format uses .dic.dz file witch is gzipped plain .dic file

2. Compressing .dic file with gzip is not enough for creating compressed dictionary. As I can guess an index should be regenerated somehow
to point to the correct places of the compressed file. But I've not yet try to find out how it should work.
