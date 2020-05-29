# For and While Loop Vectorization

### An example of for and while loop that computes the addition of two arrays of integers

```c
// for loop version

void example_for( int *restrict out, int *restrict a, int *restrict b, int N) {
    for (int i=0; i<N; i++) {
        out[i] = a[i] + b[i];
    }
}
```

```c
// while loop version

void example_while( int *restrict out, int *restrict a, int *restrict b, int N) {
    while (N > 0) {
       *out = *a + *b;
        a++;
        b++;
        out++;
        N = N - 1;
    }
}

// both of them have the same functionality
```

### For/While Loop SVE Vectorization

```bash
      MOV w3, w3
      MOV x9, #0                        # load i as 0

      WHILELT p1.s, x9, x3
      B.NONE .L_return

  .L_loopStart:
      LD1W z1.s, p1/Z, [x1, x9, LSL #2]
      LD1W z2.s, p1/Z, [x2, x9, LSL #2]
      ADD z1.s, p1/M, z1.s, z2.s
      ST1W z1.s, p1, [x0, x9, LSL #2]
      INCW x9                           # i++
      WHILELT p1.s, x9, x3
      B.FIRST .L_loopStart              # check the LSB

  .L_return:
      RET
```

