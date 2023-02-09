---
title: "Matrix multiply in x86"
date: 2023-01-15T16:24:27+01:00
draft: true
---

Let's learn x86_64 by writing a matrix multiplier.

## Matrix multiplication (mm)

As a reminder, matrix multiplication \\(AB\\), works by taking the dot product of every row in \\(A\\) with every column in \\(B\\).

![Matrix multiplication illustration](/posts/Matrix_multiplication_diagram_2.svg)

If \\(A\\) has \\(N\\) rows and \\(K\\) columns, and \\(B\\) has \\(K\\) rows and \\(M\\) columns, then \\(AB\\) has \\(N\\) rows and \\(M\\) columns. If they do not share this dimension \\(K\\) then matrix multiplication is not applicable. As pseudo-code mm boils down to

```
for i = 0 to N
	for j = 0 to M
		for k = 0 to K
			C[i, j] += A[i, k] * B[k, j]
```

This requires \\(O(N^3)\\) operations for matrices with largest dimension \\(N\\). There are theoretically faster algorithms but in practice they have large constant factors. Therefore, we stick to the simple algorithm.

## Optimisations

We can start by making our algorithm cache-aware. Since main memory is linear, our matrices are "flattened" in row-major order. Below is an example of such a flattening.

\\[
		\begin{bmatrix}
    		1 & 3 & 0\\\
    		-2 & 4 & 2
  		\end{bmatrix} \quad \text{ is stored as }\quad \begin{bmatrix}1 & 3 & 0 & -2 & 4 & 2\end{bmatrix}
\\]

We basically place the matrix row by row in the memory. When we read from a memory location we also cache nearby memory, so iterating over a column may result in many cache misses if the rows are large enough. Therefore, when using row-major order, it is best to only deal with row operations. We can modify our algorithm to do so.

```
for i = 0 to N
	for j = 0 to M
		for k = 0 to K
			C[i, k] += A[i, j] * B[j, k]
```

Now we turn our attention to leveraging parallelism. If we "unroll" the inner-most part of the loop a bit.

```
for i = 0 to N
	for j = 0 to M
		for k = 0 to K/4
			C[i, k] += A[i, j] * B[j, k]
			C[i, k+1] += A[i, j] * B[j, k+1]
			C[i, k+2] += A[i, j] * B[j, k+2]
			C[i, k+3] += A[i, j] * B[j, k+3]
```

We can vectorise the four operations, \\(A[i,j]\cdot\begin{pmatrix}
  	B[j, k]\\\
    B[j, k+1]\\\
    B[j, k+2]\\\
    B[j, k+3]
  \end{pmatrix}\\), to exploit SIMD instructions.


## System stats

Using the `lscpu` command I get information about my system architecture that you can use for reference.

```
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              2
Model name:                      Intel(R) Core(TM) i7-8550U CPU @ 1.80GHz
CPU MHz:                         876.059
CPU max MHz:                     4000.0000
CPU min MHz:                     400.0000
L1d cache:                       128 KiB
L1i cache:                       128 KiB
L2 cache:                        1 MiB
L3 cache:                        8 MiB
```

