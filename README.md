# Microcorruption Writeups

This contains my writeups for the challenges on [microcorruption.com](https://microcorruption.com/).

## Instruction Set

The instruction set used on Microcorruption is MSP430, which can be found [here](https://www.ti.com/sc/docs/products/micro/msp430/userguid/as_5.pdf). The PDF is also stored in this repository for posterity's sake.

### Properties

Some properties of this instruction set:

- **Endianness**: Little endian
- **Word size**: 16 bits (2 bytes)

### Differences with x86

Notable differences between this instruction set and the x86 set includes (WIP, will be continually updated):

1.  **Inversion of the instruction operands**  
    In x86, instructions such as `mov eax,ebx` or `add eax,ebx` indicates that the `eax` register is updated with the value of the final computation; On the other hand, the MSP430 uses `src,dest` ordering instead.

## License

[MIT](LICENSE.md)
