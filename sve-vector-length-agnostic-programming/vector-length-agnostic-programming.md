# Vector Length Agnostic Programming

## Examples on VLA Programming

> The goal of SVE is to allow the same program binary to be run on any implementation of the architecture, which might implement different vector lengths

> The SVE features requires a new programming style, called Vector Length Agnostic \(VLA\).

For example, we want to do vector-vector-add between two vectors `b[]` and `c[]`, and then store the result in `a[]`. There are three versions of these programs:

1. Traditional method
2. Programs on ARM Neon
3. Programs on SVE

### Traditional method

```c
void example01(int *restrict a, const int *b, const int *c, long N)
{
  long i;
  for (i = 0; i < N; ++i)
    a[i] = b[i] + c[i];
}
```

### Programs on ARM Neon

```c
// it is a vectorized version of the traditional SIMD version
void example01_neon(int *restrict a, const int *b,
                    const int *c, long N)
{
  long i;
  // vector loop
  for (i = 0; i < N - 3; i += 4) {
    int32x4_t vb = vld1q_s32(b + i);  // vector load
    int32x4_t vc = vld1q_s32(c + i);  // vector load
    int32x4_t va = vaddq_s32(vb, vc); // vvadd
    vst1q_s32(a + i, va);             // vector store
  }
  // loop tail
  for (; i < N; ++i)
    a[i] = b[i] + c[i];
}

```

### Programs on SVE

```bash
    # x0 is 'a', 
    # x1 is 'b', 
    # x2 is 'c', 
    # x3 is 'N', 
    # x4 is 'i'

    mov     x4, 0  # set 'i=0'
    b       cond   # branch to 'cond'
loop_body:
    ld1w    z0.s, p0/z, [x1, x4, lsl 2] # load vector z0 from address 'b + i'
    ld1w    z1.s, p0/z, [x2, x4, lsl 2] # same, but from 'c + i' into vector z1
    add     z0.s, p0/m, z0.s, z1.s      # add the vectors
    st1w    z0.s, p0, [x0, x4, lsl 2]   # store vector z0 at 'a + i'
    incw    x4                          # increment 'i' by number of words in a vector
cond:
    whilelt p0.s, x4, x3   # build the loop predicate p0, as p0.s[idx] = (x4+idx) < x3
                           # it also sets the condition flags
    b.first    loop_body   # branch to 'loop_body' if the first bit in the predicate
                           # register 'p0' is set
    ret
```

The SVE assembler code can be transferred to the pseudo C code:

```c
while( /* cond */ ) {
  /* loop */
}
```

Here is the introduction of the condition:

```bash
    # How whilelt works:
    
    # It sets each predicate elements in the predicate register p0 if we have:
    # p0.s[idx] := (x3 + idx) < x4   (x3 and x4 hold i and N respectively)
    # This is an example of 256-bit SVE implementation for N=7
    # P0 = [0000 0001 0001 0001 0001 0001 0001 0001]
    #          7    6    5    4    3    2    1    0    32-bit lanes 'x'
    
    # The whilelt not only builds the predicate p0, but also sets the condition flags
    # b.first instruction reads those flags and decides whether or not to branch
    # to the loop_body label.
    # b.first checks if the first (LSB) element of p0 is set to true which means there
    # are any further elements to process in the next iteration of the loop.
```

Here is the introduction of the loop body:

```bash
ld1w  # it uses "register plus register address mode"
incw  # increment the index x4 according to the current SVE vector length!!
```

## More on Predication

> What about the `if` statement in the vectorized loops?

Now we have a if condition inside the for loop. Here is the C program version:

```c
void example02(int *restrict a, const int *b, const int *c, long N,
                      const int *d)
{
  long i;
  for (i = 0; i < N; ++i)
    if (d[i] > 0)
      a[i] = b[i] + c[i];
}
```

### Programs on SVE

```bash
# x0 is 'a', 
# x1 is 'b', 
# x2 is 'c', 
# x3 is 'N', 
# x4 is 'd', 
# x5 is 'i'

    mov     x5, 0 # set 'i = 0'
    b       cond
loop_body:
    ld1w    z4.s, p0/z, [x4, x5, lsl 2] # load a vector from 'd + i'
    cmpgt   p1.s, p0/z, z4.s, 0         # compare greater than zero
                                        # p1.s[idx] = z4.s[idx] > 0
    # from now on all the instructions depending on the 'if' statement are
    # predicated with 'p1'
    ld1w    z0.s, p1/z, [x1, x5, lsl 2]
    ld1w    z1.s, p1/z, [x2, x5, lsl 2]
    add     z0.s, p1/m, z0.s, z1.s
    st1w    z0.s, p1, [x0, x5, lsl 2]
    incw    x5
cond:
    whilelt p0.s, x5, x3
    b.ne    loop_body
    ret
```

## Merging and Zeroing Predication

```bash
p1/z  # zeroing the p1 predication register
p1/m  # merging the p1 predication register

# zeroing: sets the inactive lanes to zero
# merging: does not change the inactive lanes
```

