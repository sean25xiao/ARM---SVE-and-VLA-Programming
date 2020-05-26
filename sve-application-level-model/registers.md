# Registers

### Vector Registers

![Figure 1: SVE Vector Registers](../.gitbook/assets/image.png)

There are 32 scalable vector registers, `z0 ~ z31`

* Each of their length is a multiple of 128 bits with minimum 128 bits and maximum 2048 bits.
* Each vector can be divided by several elements with 128 bits \(.Q\), 64 bits\(.D\), 32 bits \(.S\), 16 bits\(.H\), or even 8 bits\(.B\).

