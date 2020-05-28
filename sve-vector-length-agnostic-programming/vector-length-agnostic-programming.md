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

### Programs on ARM Neon

```c
void example01_neon(int *restrict a, const int *b,
                    const int *c, long N)
{
  long i;
  // vector loop
  for (i = 0; i < N - 3; i += 4) {
    int32x4_t vb = vld1q_s32(b + i);
    int32x4_t vc = vld1q_s32(c + i);
    int32x4_t va = vaddq_s32(vb, vc);
    vst1q_s32(a + i, va);
  }
  // loop tail
  for (; i < N; ++i)
    a[i] = b[i] + c[i];
}

```

