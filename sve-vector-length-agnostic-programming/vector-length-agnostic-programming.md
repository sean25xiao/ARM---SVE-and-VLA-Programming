# Vector Length Agnostic Programming

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

### 

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

```c
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

