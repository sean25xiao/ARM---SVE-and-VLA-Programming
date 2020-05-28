# Registers

### Vector Registers

![Figure 1: SVE Vector Registers](../.gitbook/assets/image.png)

There are 32 scalable vector registers, `z0 ~ z31`

* Each of their length is a multiple of 128 bits with minimum 128 bits and maximum 2048 bits.
* Each vector can be divided by several elements with 128 bits \(.Q\), 64 bits\(.D\), 32 bits \(.S\), 16 bits\(.H\), or even 8 bits\(.B\).
* The element size is encoded in the opcode.
* The `[127:0]` of `z0 ~ z31` are shared with the SIMD&FP registers, `V0 to V31` in AArch64

![Figure 2: 128-bit SIMD&amp;FP registers in AArch64](../.gitbook/assets/image%20%281%29.png)

### Predicate Registers

![Figure 3: 32-bit predicate register of an 256-bit vector register](../.gitbook/assets/image%20%282%29.png)

* Each one byte \(eight bits\) of the vector registers have their own one-bit predicate register; therefore, the length of the predicate registers is one-eighth of the size of the vector register.
* Predicate register length is a multiple of 16
* They can be subdivided into 1 bit, 2 bits, 4 bits, or 8 bits.
* For all instruction except those listed in Predicate Permute, the last bit in each element of the predicate register is focused. The other bits will be ignored and set to zero on writes.
* If the last bit is 1, the predicate element if `TRUE`; if it is 0, then the predicate element is `FALSE`

> An instruction which supports the predication is called a predicated instruction.

> The predicate register that is used to determine the Active elements of a predicated instruction is known as the Governing predicate for that instruction.

* Many predicated instructions are only supported to use `P0 ~ P7`.
* When the governing predicate element is `TRUE`, then the corresponding vector or predicate register element is Active and is processed by the instruction; if it is `FALSE`, it is inactive and takes no part in the operation of the instruction.

> When a predicated instruction writes to a vector or predicate destination registers and the Inactive elements in the destination are set to zero, this is known as zeroing predication.

> When a predicated instruction writes to a vector or predicate destination registers and the Inactive elements in the destination retain their previous value, this is known as merging predication.

![Figure 4: Predicate registers can be interpreted differently](../.gitbook/assets/image%20%283%29.png)

