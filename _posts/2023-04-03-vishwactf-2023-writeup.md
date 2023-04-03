---
layout: post
title: "VishwaCTF 2023 Writeup"
date: 2023-04-03
categories: ctf
excerpt_separator: <!--more-->
---

Writeup for VishwaCTF 2023 challenges: ReverseHill, WednesdayThursdayFriday, PhiCalculator. Finished as 188th, with 8 flags and 1190 points. <!--more-->


### Reverse Hill (300 Points)

Start by decompiling the ELF with IDA.

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int result; // eax
  int k; // [rsp+4h] [rbp-Ch]
  int j; // [rsp+8h] [rbp-8h]
  int i; // [rsp+Ch] [rbp-4h]

  for ( i = 0; i <= 5; ++i )
    function1((unsigned int)i);
  result = puts("\nFLAG IS:");
  for ( j = 0; j <= 5; ++j )
  {
    for ( k = 0; k <= 2; ++k )
      result = printf("%c ", (unsigned int)d[3 * j + k]);
  }
  return result;
}
```

The binary prints out contents of `d` as the flag. We can deduce from the two for-loops that `d` is a 2D-array, or a matrix, with dimension 6 x 3. <br>
However, executing the binary prints out random characters, so we explore further by looking at `function1`.

```c
int __fastcall function1(int a1)
{
  double v1; // xmm0_8
  double v2; // xmm0_8
  float v4[3]; // [rsp+10h] [rbp-20h]
  int n; // [rsp+1Ch] [rbp-14h]
  int i; // [rsp+20h] [rbp-10h]
  int m; // [rsp+24h] [rbp-Ch]
  int k; // [rsp+28h] [rbp-8h]
  int j; // [rsp+2Ch] [rbp-4h]

  function2();
  for ( i = 0; i <= 2; ++i )
    v4[i] = *(float *)&in[3 * a1 + i];
  putchar(10);
  for ( j = 0; j <= 2; ++j )
    decrypt[j] = 0.0;
  for ( j = 0; j <= 2; ++j )
  {
    for ( k = 0; k <= 0; ++k )
    {
      for ( m = 0; m <= 2; ++m )
        decrypt[k + (__int64)j] = (float)(v4[k + (__int64)m] * b[3 * j + m]) + decrypt[k + (__int64)j];
    }
  }
  for ( n = 0; n <= 2; ++n )
  {
    v1 = fmod(decrypt[n], 26.0);
    v2 = round(v1);
    d[3 * a1 + n] = (int)(v2 + 97.0);
  }
  return putchar(10);
}
```

`function1` does indeed manipulate the contents in `d`, at the bottom of its code. We see magic values *97* (ascii value of 'a') and *26* (size of the alphabet), confirming our goal to try and find out what the contents of `d` are.
<br>
We can't find out much from `function1` alone, so we continue our search into `function2`.

```c
int function2()
{
  float *v0; // rax
  float v2; // [rsp+4h] [rbp-Ch]
  int j; // [rsp+8h] [rbp-8h]
  int n; // [rsp+8h] [rbp-8h]
  int i; // [rsp+Ch] [rbp-4h]
  int k; // [rsp+Ch] [rbp-4h]
  int m; // [rsp+Ch] [rbp-4h]

  v2 = 0.0;
  for ( i = 0; i <= 2; ++i )
  {
    for ( j = 0; j <= 2; ++j )
      c[3 * i + j] = *(float *)&a[3 * i + j];
  }
  LODWORD(v0) = function3();
  for ( k = 0; k <= 2; ++k )
  {
    v0 = c;
    v2 = (float)((float)((float)(c[(k + 2) % 3 + 6] * c[(k + 1) % 3 + 3])
                       - (float)(c[(k + 1) % 3 + 6] * c[(k + 2) % 3 + 3]))
               * c[k])
       + v2;
  }
  for ( m = 0; m <= 2; ++m )
  {
    for ( n = 0; n <= 2; ++n )
      b[3 * m + n] = (float)((float)(c[3 * ((n + 2) % 3) + (m + 2) % 3] * c[3 * ((n + 1) % 3) + (m + 1) % 3])
                           - (float)(c[3 * ((n + 2) % 3) + (m + 1) % 3] * c[3 * ((n + 1) % 3) + (m + 2) % 3]))
                   / v2;
    LODWORD(v0) = putchar(10);
  }
  return (int)v0;
}
```

In `function2` we start to get a hint of what is going on. The contents of `a` (Matrix with dimension 3x3) is defined at compile time, and is copied into `c`.<br>
A call to `function3` is made (which we explore later), and `b` (used in `function1`) is defined here.<br>
Tracing the next two for-loops reveals:
- `v2` is the determinant of the matrix `c`
- `b` is a new matrix calculated from `c` using `v2`
<br>

At this point, given the phrases Reverse **Hill**, `decrypt` in `function1`, Matrices and determinants, we can reasonably guess that this is a [hill cipher](https://en.wikipedia.org/wiki/Hill_cipher) decryptor, and matrix `b` is the inverse matrix of `c`.

Now we take a look at `function3`

```c
float *function3()
{
  unsigned int v0; // eax
  float v1; // xmm0_4
  float *result; // rax
  int j; // [rsp+0h] [rbp-10h]
  int i; // [rsp+4h] [rbp-Ch]
  int v5; // [rsp+8h] [rbp-8h]
  int v6; // [rsp+Ch] [rbp-4h]

  v0 = time(0LL);
  srand(v0);
  for ( i = 0; i <= 2; ++i )
  {
    for ( j = 0; j <= 2; ++j )
    {
      v6 = rand() % 3;
      v5 = rand() % 3;
    }
  }
  v1 = (double)rand() / 1.0e10;
  result = c;
  c[3 * v6 + v5] = v1;
  return result;
}
```

`function3` replaces an element at a randomly chosen index (`3 * v6 + v5`) with a random value (`rand() / 1.0e10`).<br>
This is probably the function that is messing up the decryption process, causing rubbish to be printed out when executed.<br>
We can either patch this or manually skip it by using a debugger, to get our flag printed out:

```bash
$> m a t r i x t h e w a y a r o u n d
```
In flag format:

`VishwaCTF{matrix_the_way_around}`





