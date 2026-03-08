# ROMCat
**A catalog based ROM filesystem**  

## Used terms

 - **Block:**
  - **System block:**
   - **Header:**: is the beginning of the system block, contains the magic word,
     version number, and ROM size in blocks.
   - **File Descriptor Table:** contains the file name, date/timestamp, starting
     block, size, attributes, metadata pointer, and length. The first file entry
     is the volume label, it has no size and no starting block.
   - **Metadata area:** contains optional additional information related to the
     file. Its size is not fixed, only the total size is finite, this depends on
     the capacity of the EPROM used.
   - **Checksum table:** contains a checksum for each data block, except for the
     system block, which is always 00h. This is a 8-bit Modulo 256 sum value of
     the bytes.
  - **Data block:** is used to store the contents of files. The contents of
     files start at the block boundary, but can extend into the next block(s).

## 1. Concept

- **Goal:** To provide a compact ROM-based file system that allows direct access
  to files using a single descriptor table and contiguous data storage,
  minimizing lookup complexity and runtime processing on resource-limited
  systems.

## 2. Blocks in the EPROM

|from| to |sign |function    |note                           |
|----|----|-----|------------|-------------------------------|
|0000|07FF|BLK00|system block|                               |
|0800|0FFF|BLK01|data block  |                               |
|1000|17FF|BLK02|data block  |                               |
|1800|1FFF|BLK03|data block  |last block of the 2764 (8 kB)  |
|2000|27FF|BLK04|data block  |                               |
|2800|28FF|BLK05|data block  |                               |
|3000|37FF|BLK06|data block  |                               |
|3800|3FFF|BLK07|data block  |last block of the 27128 (16 kB)|
|4000|47FF|BLK08|data block  |                               |
|(..)|(..)|(..) |data block  |                               |
|7000|77FF|BLK14|data block  |                               |
|7800|7FFF|BLK15|data block  |last block of the 27256 (32 kB)|
|8000|87FF|BLK16|data block  |                               |
|(..)|(..)|(..) |data block  |                               |
|F000|F7FF|BLK30|data block  |                               |
|F800|FFFF|BLK31|data block  |last block of the 27512 (64 kB)|

### 3. Structure of the system block (BLK00)

**Common header area**  

|from| to |sign    |area    |function        |note                  |
|----|----|--------|--------|----------------|----------------------|
|0000|0001|HDR_MAWD|header  |magic word      |                      |
|0002|    |HDR_VERS|header  |version         |high nibble.low nibble|
|0003|    |HDR_BLKN|header  |number of blocks|                      |

**Additional areas in the 2764 EPROM**  

|from| to |sign    |area                   |function              | note              |
|----|----|--------|-----------------------|----------------------|-------------------|
|0004|001C|FDT_FL00|file descriptor table  |volume record         |                   |
|001D|0035|FDT_FL01|file descriptor table  |file record           |                   |
|0036|004E|FDT_FL02|file descriptor table  |file record           |                   |
|004F|0067|FDT_FL03|file descriptor table  |file record           |                   |
|0068|07FB|MDT_FLnn|metadata area          |metadata of the files |no fixed boundaries|
|07FC|07FF|CSM_BLnn|checksum area (4 block)|checksum of the blocks|increases to bottom|

**Additional areas in the 27128 EPROM**  

|from| to |sign    |area                   |function              | note              |
|----|----|--------|-----------------------|----------------------|-------------------|
|0004|001C|FDT_FL00|file descriptor table  |volume record         |                   |
|001D|0035|FDT_FL01|file descriptor table  |file record           |                   |
|0036|004E|FDT_FL02|file descriptor table  |file record           |                   |
|004F|0067|FDT_FL03|file descriptor table  |file record           |                   |
|(..)|(..)|FDT_FLnn|file descriptor table  |file record           |                   |
|00B3|00CB|FDT_FL07|file descriptor table  |file record           |                   |
|00CC|38D6|MDT_FLnn|metadata area          |metadata of the files |no fixed boundaries|
|38D7|38FF|CSM_BLnn|checksum area (8 block)|checksum of the blocks|increases to bottom|

**Additional areas in the 27256 EPROM**  

|from| to |sign    |area                    |function              | note              |
|----|----|--------|------------------------|----------------------|-------------------|
|0004|001C|FDT_FL00|file descriptor table   |volume record         |                   |
|001D|0035|FDT_FL01|file descriptor table   |file record           |                   |
|0036|004E|FDT_FL02|file descriptor table   |file record           |                   |
|004F|0067|FDT_FL03|file descriptor table   |file record           |                   |
|(..)|(..)|FDT_FLnn|file descriptor table   |file record           |                   |
|0193|01AB|FDT_FL15|file descriptor table   |file record           |                   |
|01AC|7FAF|MDT_FLnn|metadata area           |metadata of the files |no fixed boundaries|
|7FB0|7FFF|CSM_BLnn|checksum area (16 block)|checksum of the blocks|increases to bottom|

**Additional areas in the 27512 EPROM**  

|from| to |sign    |area                    |function              | note              |
|----|----|--------|------------------------|----------------------|-------------------|
|0004|001C|FDT_FL00|file descriptor table   |volume record         |                   |
|001D|0035|FDT_FL01|file descriptor table   |file record           |                   |
|0036|004E|FDT_FL02|file descriptor table   |file record           |                   |
|004F|0067|FDT_FL03|file descriptor table   |file record           |                   |
|(..)|(..)|FDT_FLnn|file descriptor table   |file record           |                   |
|0323|033B|FDT_FL31|file descriptor table   |file record           |                   |
|033C|FF5F|MDT_FLnn|metadata area           |metadata of the files |no fixed boundaries|
|FF60|FFFF|CSM_BLnn|checksum area (32 block)|checksum of the blocks|increases to bottom|

### 4. Structure of the file descriptor table

|size   |function                    | note                              |
|------:|----------------------------|-----------------------------------|
|11 byte|filename with extension     |                                   |
| 4 byte|date/time in DOS format     |                                   |
| 1 byte|number of the starting block|                                   |
| 2 byte|filesize in byte            |                                   |
| 1 byte|user/attribute              |CP/M user number and DOS attributes|
| 2 byte|metadata address            |                                   |
| 2 byte|metadata length             |                                   |
| 1 byte|reserved                    |                                   |

## 5. Examples

See _example.bin_ file.

## 6. Implementations

The following assembly implementations are under development and will be available soon as linkable .REL files:

- _Intel 8080_: targeted for CP/M OS,
- _Intel 8086_: targeted for DOS,
- _Zilog Z80_: targeted for CP/M OS.

This collection of routines only handles read processing between I/O operations.

Any other implementations and additions are welcome and will be published.
