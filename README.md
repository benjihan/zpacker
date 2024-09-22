# zpacker - very simple LZ77-based compression

This is LZ77-based compression code with a focus on small files, and
simplicity of decompression. Packing (and depacking) code is done in C
language, and depacking in 68000 assembly language is also provided.

## Features

As it is focused on small files, no fancy features like 32-bit offsets
or chunk sizes. The packer will behave slightly better than lz4 on
small files, but significantly worse on larger files.

## File format

The compressed file format is a succession of two types of chunks:

The two MSB (bit#7 and bit#6) of the header byte selects the type of
chunk: 0xC0 (both bits set) means it is a _litteral chunk_ else it is
an _offset chuck_.

 - _Literal chunks_ : Chunks containing not packed data. The number of
   data bytes to copy is encoded in the remaining 6-bits (bit#5 to
   bit#0) with a range of 1 to 64. It is encoded with `191+size`
   yielding a value from 192 to 255. The data follow directly the
   header byte.
 
- _Offset chunks_ : Chunks describing a repetition of previous
  data. The number of byte to copy is encoded in the header byte with
  respect of the _litteral chunks_ constraint. The range is 4
  to 195. It is encoded as `195-size` yielding a value from 191 down
  to 0.  The next byte is the offset from -1 to -256 in 2's complement
  encoding.
 
It may seem complicated for no reason but it was specially crafted to
help reduce the size of the m68k unpack routine.
