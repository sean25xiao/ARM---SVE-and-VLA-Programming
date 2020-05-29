# Do-while Loop SVE Vectorization

Here is the Do-while loop that computes the sum of the first N squares:

```c
void example_sum_squares( int N, int * sum) {
    int res = 0;
    if (N > 0) {
        do {
            res += N*N;
            N--;
        } while (N > 0);
    }
    *sum = res;
}
```

```bash
       PTRUE p1.s
       MOV z5.s, #0
       MOVI d6, #0

       INDEX z0.s, x0, #-1         # partition the N as vector=
       CMPGT p0.s, p1/Z, z0.s, #0  # set the p0
       B.NONE .L_result

  .L_loopStart:
      MLA z5.s, p0/M, z0.s, z0.s
      DECW z0.s
      CMPGT p0.s, p1/Z, z0.s, #0
      B.FIRST .L_loopStart

      UADDV d6, p1, z5.s

  .L_result:
      STR s6, [x1]
      RET
```

* `INDEX` sets the starting input elements for each vector lane. It uses `#-1` to map the `N--` in the C program
* `CMPGT` compares the elements of N in `z0` and set the loop process-governing predicate `p0`.
* In `line 9~13`, the governing predicate controls the active lanes that contribute to the partial sums, computed with instruction `MLA (Multi-Add to Accumulator)`
* DECW can compute the next group of input elements of new N values. To obtain the input element of that lane for the next loop iteration, the current input element in each vector lane is decremented by the number of `32-bit` elements in the implemented vector length.
* If at least one positive input element exists, the next loop iteration occurs. Otherwise, the loop exists. This is done by `B.FIRST`.
* UADDV sums up all the partial sums according to the predicate register `p1`. It locates outside the loop.

he SVE vector length agnostic vectorization code can be rewritten in C with the [ARM C language extension \(ACLE\) for SVE](https://developer.arm.com/docs/100987/latest/arm-c-language-extensions-for-sve):

```c
void example_sum_squares( int N, int * sum) {
2       svbool_t pred_N;
3       svint32_t svN_tmp;
4       svbool_t p_all = svptrue_b32();
5       svint32_t acc = svdup_s32(0);
6
7       if (N > 0) {
8           svN_tmp = svindex_s32( N, -1);
9           pred_N = svcmpgt( p_all, svN_tmp, 0);
10
11          do {
12              acc = svmla_m( pred_N, acc, svN_tmp, svN_tmp);
13              svN_tmp = svsub_x( p_all, svN_tmp, svcntw());
14              pred_N = svcmpgt( p_all, svN_tmp, 0);
15          } while ( svptest_first( p_all, pred_N));
16      }
17
18      *sum = (int) svaddv( p_all, acc);
19  }
```

