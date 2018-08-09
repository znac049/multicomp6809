ROM images for programming into the FPGA 8Kbyte ROM. A ROM image must:

* be in Intel .hex format
* be assembled to execute from $E000
* include the reset and other exception vectors
* have an entry point corresponding to the value of the reset vector
* despite the execution address of $E000, the .hex file must contain addresses $0000-$1FFF

The script 'builtit' can be used to assemble a file in order to produce a suitable .hex file. For example:

    $ ./buildit foo.asm

Creates these files:

    foo.s19        -- executable in srecord format
    foo.lst        -- assembly listing
    foo.hex        -- executable in hex format
    foo.fixed_hex  -- executable in hex format, rebased from $e000 to $0000
                      ready to program into an FPGA ROM

There are 2 'production' ROMs for multicomp6809 and several 'debug' ROMs. The 'production' ROMs are:

    CAMELFORTH_2KRAM.hex
    M6809_CAMELFORTH_ROM.cmp
    M6809_CAMELFORTH_ROM.cnx
    M6809_CAMELFORTH_ROM.qip
    M6809_CAMELFORTH_ROM.vhd

    EXT_BASIC_NO_USING.hex
    M6809_EXT_BASIC_ROM.cmp
    M6809_EXT_BASIC_ROM.qip
    M6809_EXT_BASIC_ROM.vhd

The .hex file is the ROM content and the other files are required by the Altera
tool chain to instance a ROM in the design. The BASIC ROM is Grant's port of
Microsoft BASIC. The CAMELFORTH ROM is my port of Brad Rodruigez's 6809 Camel
FORTH.

My multicomp6809 is set up to use the CAMELFORTH ROM. My SDcard image includes
the Microsoft BASIC and FORTH provides the work BASIC to load and start it.

The '_2KRAM' in the name of the CAMELFORTH image refers to RAM address space
that the code is set up to run with.

I'm not sure why there is a .cnx file for the CAMELFORTH ROM and not for the
other ROM; it doesn't seem to be needed.

## Rebuilding the BASIC ROM

Start with basic.asm (Grant's original source)

    $ ./buildit basic.asm

Hand-edit the basic.fixed_hex to delete 2 lines at the start so that the first line starts like this:

    :200000008

This now matches Grant's original EXT_BASIC_NO_USING.hex with the exception that
his image has the unused locations (between the end of the code and the start of
the vectors) filled, whereas the new image omits them. The FPGA flow does not care.

In order to put the BASIC image onto the SDcard, a binary image is required, which can be created like this:

    $ ../../../bin/hex2bin EXT_BASIC_NO_USING.hex > EXT_BASIC_NO_USING.bin

## Rebuilding the CAMELFORTH ROM

Start with 6809M.HEX, the file generated by the CAMEL FORTH compilation process.

   $ ../../../bin/hex_addr_remap e000 6809M.HEX > CAMELFORTH_2KRAM.hex

## Other files here

* defines.asm -- IO address defines for multicomp6809