# unuzip - Decompress FreeBSD UZIP images

UZIP is a compressed image format created by `mkuzip` on FreeBSD, not
entirely dissimilar to cramfs on Linux. `unuzip` decompresses these
images to plain UFS images that can be mounted on Linux.

unuzip is written in Python 2 and has been tested on Linux and Mac OS X.

## Usage

    unuzip [-v] <in.uzip> [out.ufs]

Decompress the input file. If no output file is given, the output
filename defaults to `inputfile.ufs`. Add the `-v` flag to get details
about each block as they are decompressed.

Note that `unuzip` will silently overwrite the output file.

Once the file is decompressed, you can mount it on a Linux system using
a command like:

    mount -o loop,ro,ufstype=ufs2 out.ufs /mnt/somewhere

# Why?

As far as I could find, no tool exists to decompress UZIP images. I
don't have any FreeBSD systems handy, but Linux can easily mount plain
UFS file systems. Thus this tool was born.

# UZIP file format

UZIP does not appear to be formally documented anywhere, so for the
benefit of others interested in this format, this section is a brief
overview of how the file is structured. All integers are big endian.

    Magic:  128 bytes
    Header: 8 bytes (uint32_t block_size, uint32_t total_blocks)
    TOC:    8 bytes * (num_blocks + 1)
    Compressed blocks

UZIP magic is a 128 byte string that also makes the file a valid shell
script that will mount the image on FreeBSD. For version 2 images, the
magic string is:

    #!/bin/sh
    #V2.0 Format
    m=geom_uzip
    (kldstat -m $m 2>&-||kldload $m)>&-&&mount_cd9660 /dev/`mdconfig -af $0`.uzip $1
    exit $?

Version 3 images change the second line to `#L3.0 Format`, but this tool
does not support such images.

The header declares the block size (size of decompressed blocks) and
total number of blocks. Block size must be a multiple of 512 and
defaults to 16384 in `mkuzip`.

The TOC is a list of `uint64_t` offsets into the file for each block. To
determine the length of a given block, read the next TOC entry and
subtract the current offset from the next offset (this is why there is
an extra TOC entry at the end).

Each block is compressed using zlib. A standard zlib decompressor will
decode them to a block of size `block_size`.

# About
`unuzip` was written by Mike Ryan of [ICE9 Consulting](https://ice9.us).
Please report any issues on the [unuzip GitHub](https://github.com/mikeryan/unuzip).

`unuzip` is not part of FreeBSD and shares no code with FreeBSD.
