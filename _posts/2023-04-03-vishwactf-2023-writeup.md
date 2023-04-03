---
layout: post
title: "VishwaCTF 2023 Writeup"
date: 2023-04-03
categories: ctf
excerpt_separator: <!--more-->
---

Finished as 188th, with 8 flags and 1190 points.<br>
Writeup for VishwaCTF 2023 challenges:
- [Reverse Hill (300 Points, 20 Solves)](#reverse-hill-300-points-20-solves)
- [WednesdayThursdayFriday (283 Points, 86 Solves)](#wednesday-thursday-friday-283-points-86-solves)
- [PhiCalculator (136 Points, 322 Solves)](#phi-calculator-136-points-322-solves)<!--more-->

<br>

## Reverse Hill (300 Points, 20 Solves)

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

`function3` replaces an element of `c` at a randomly chosen index (`3 * v6 + v5`) with a random value (`rand() / 1.0e10`).<br>
This is the function that is messing up the inverse matrix `b` and hence the decryption process, causing rubbish to be printed out when executed.<br>
We can either patch this or manually skip it by using a debugger, to get our flag printed out:

```bash
$> m a t r i x t h e w a y a r o u n d
```

Flag: `VishwaCTF{matrix_the_way_around}`

<br>
<br>
<br>

## Wednesday Thursday Friday (283 Points, 86 Solves)

Start by looking at the decompiled main in IDA

```c
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  char *s; // [rsp+28h] [rbp-8h]

  if ( a1 == 2 )
  {
    s = a2[1];
    if ( strlen(s) == 34
      && s[3] + s[4] + s[1] + s[7] - s[8] * s[2] * s[6] * s[5] - s[11] - s[9] - s[10] == -52316790
      && s[3] - s[4] - s[6] + s[9] + s[8] * s[11] * s[10] - s[2] + s[5] + s[7] * s[12] == 285707
      && s[11] + s[10] * s[4] + s[3] - s[12] * s[7] - s[13] - s[5] * s[9] * s[6] + s[8] == -797145
      && s[4] + s[12] - s[7] * s[11] - s[9] - s[5] * s[6] - s[14] - s[8] * s[13] * s[10] == -289275
      && s[13] + s[14] + s[7] + s[6] - s[12] - s[15] * s[11] - s[5] + s[8] * s[10] * s[9] == 666868
      && s[12] + s[15] * s[16] + s[11] + s[13] - s[10] + s[6] * s[8] - s[7] - s[9] + s[14] == 9837
      && s[7] + s[11] - s[8] + s[16] * s[13] - s[17] - s[14] - s[9] + s[10] * s[15] - s[12] == 9858
      && s[17] + s[12] + s[9] - s[18] - s[8] - s[15] + s[16] + s[11] * s[14] * s[13] - s[10] == 296504
      && s[11] * s[13] * s[18] * s[16] - s[17] - s[10] + s[9] + s[15] * s[12] - s[19] - s[14] == 10963387
      && s[17] + s[16] + s[20] + s[12] - s[14] * s[18] * s[15] * s[19] - s[13] - s[11] - s[10] == -65889660
      && s[16] - s[19] - s[15] + s[11] * s[13] + s[18] + s[21] * s[12] + s[14] + s[17] * s[20] == 13340
      && s[18] * s[16] + s[17] * s[15] - s[20] - s[12] - s[19] * s[14] + s[22] + s[13] * s[21] == 4641
      && s[15] + s[20] + s[18] + s[21] + s[13] * s[19] - s[22] - s[16] - s[14] + s[17] * s[23] == 6428
      && s[19] * s[24] + s[15] * s[20] + s[16] * s[14] + s[23] - s[18] * s[21] - s[22] * s[17] == 7851
      && s[19] + s[24] + s[22] + s[21] + s[25] + s[16] + s[18] + s[20] * s[23] - s[15] + s[17] == 2997
      && s[17] * s[23] + s[20] * s[25] - s[16] + s[26] * s[21] - s[24] + s[22] * s[19] * s[18] == 342425
      && s[20] + s[26] + s[24] * s[17] + s[27] * s[22] * s[25] - s[21] - s[19] * s[18] + s[23] == 243251
      && s[24] + s[22] + s[25] * s[21] - s[28] - s[19] - s[26] * s[27] * s[20] - s[23] + s[18] == -434772
      && s[28] + s[19] + s[25] + s[29] - s[24] - s[21] - s[23] + s[27] - s[22] * s[26] + s[20] == -4957
      && s[21] + s[30] + s[26] + s[22] * s[23] - s[29] + s[20] - s[24] * s[25] - s[27] - s[28] == -1625
      && s[22] + s[26] + s[25] + s[30] + s[23] - s[24] - s[29] - s[31] - s[21] - s[27] - s[28] == -144
      && s[29] + s[30] + s[31] - s[26] - s[25] - s[23] - s[28] - s[27] - s[22] - s[32] * s[24] == -7001
      && s[33] + s[25] - s[31] * s[23] + s[27] - s[26] * s[32] + s[30] - s[24] * s[29] - s[28] == -18763 )
    {
      printf("CORRECT :)");
    }
    else
    {
      printf("INCORRECT :(");
    }
    return 0LL;
  }
  else
  {
    printf("Usage: %s <FLAG>", *a2);
    return 1LL;
  }
}
```

We are trying to find a flag such that it has length 34, and the letters that make up the flag satisfies all the equations given above.<br>
Plugging the equations directly into `z3` will not solve the problem. After closer inspection, we find that we only have 23 equations to find 34 characters.<br>
If all characters were variables, this would be extremely difficult. But given that we lack so many equations, we probably can get more information elsewhere.<br>

The flag format for this CTF is *"VishwaCTF{}"*. This string has length 11. exactly the amount of equations we lacked. (34 - 23 == 11).<br>
Now we can directly plug in values for `s[0]` up to `s[9]` and `s[33]`, and solve with `z3`!<br>
Or maybe not.<br>
`z3` still takes way too long. Let's try to learn more about this system of equations.<br>
Let's first try to manually substitute the values in for the characters we know.

```c
      && 104 + 119 + 105 + 84 - 70 * 115 * 67 * 97 - s[11] - 123 - s[10] == -52316790
      && 104 - 119 - 67 + 123 + 70 * s[11] * s[10] - 115 + 97 + 84 * s[12] == 285707
      && s[11] + s[10] * 119 + 104 - s[12] * 84 - s[13] - 97 * 123 * 67 + 70 == -797145
      && 119 + s[12] - 84 * s[11] - 123 - 97 * 67 - s[14] - 70 * s[13] * s[10] == -289275
      && s[13] + s[14] + 84 + 67 - s[12] - s[15] * s[11] - 97 + 70 * s[10] * 123 == 666868
      && s[12] + s[15] * s[16] + s[11] + s[13] - s[10] + 67 * 70 - 84 - 123 + s[14] == 9837
      && 84 + s[11] - 70 + s[16] * s[13] - s[17] - s[14] - 123 + s[10] * s[15] - s[12] == 9858
      && s[17] + s[12] + 123 - s[18] - 70 - s[15] + s[16] + s[11] * s[14] * s[13] - s[10] == 296504
      && s[11] * s[13] * s[18] * s[16] - s[17] - s[10] + 123 + s[15] * s[12] - s[19] - s[14] == 10963387
      && s[17] + s[16] + s[20] + s[12] - s[14] * s[18] * s[15] * s[19] - s[13] - s[11] - s[10] == -65889660
      && s[16] - s[19] - s[15] + s[11] * s[13] + s[18] + s[21] * s[12] + s[14] + s[17] * s[20] == 13340
      && s[18] * s[16] + s[17] * s[15] - s[20] - s[12] - s[19] * s[14] + s[22] + s[13] * s[21] == 4641
      && s[15] + s[20] + s[18] + s[21] + s[13] * s[19] - s[22] - s[16] - s[14] + s[17] * s[23] == 6428
      && s[19] * s[24] + s[15] * s[20] + s[16] * s[14] + s[23] - s[18] * s[21] - s[22] * s[17] == 7851
      && s[19] + s[24] + s[22] + s[21] + s[25] + s[16] + s[18] + s[20] * s[23] - s[15] + s[17] == 2997
      && s[17] * s[23] + s[20] * s[25] - s[16] + s[26] * s[21] - s[24] + s[22] * s[19] * s[18] == 342425
      && s[20] + s[26] + s[24] * s[17] + s[27] * s[22] * s[25] - s[21] - s[19] * s[18] + s[23] == 243251
      && s[24] + s[22] + s[25] * s[21] - s[28] - s[19] - s[26] * s[27] * s[20] - s[23] + s[18] == -434772
      && s[28] + s[19] + s[25] + s[29] - s[24] - s[21] - s[23] + s[27] - s[22] * s[26] + s[20] == -4957
      && s[21] + s[30] + s[26] + s[22] * s[23] - s[29] + s[20] - s[24] * s[25] - s[27] - s[28] == -1625
      && s[22] + s[26] + s[25] + s[30] + s[23] - s[24] - s[29] - s[31] - s[21] - s[27] - s[28] == -144
      && s[29] + s[30] + s[31] - s[26] - s[25] - s[23] - s[28] - s[27] - s[22] - s[32] * s[24] == -7001
      && s[33] + s[25] - s[31] * s[23] + s[27] - s[26] * s[32] + s[30] - s[24] * s[29] - s[28] == -18763
```

A pattern emerges: each successive equation differs by exactly one variable.<br>
That is,<br>
The first has:  `s[10]`, `s[11]`.<br>
The second has: `s[10]`, `s[11]`, `s[12]`.<br>
The third has:  `s[10]`, `s[11]`, `s[12]`, `s[13]`.<br>
And so on.<br>

This is extremely helpful: if we know `s[10]`, we would know `s[11]` directly from the first equation.<br>
Knowing `s[10]` and `s[11]` now, we would know `s[12]` directly from the second equation.<br>
And so on.<br>

Given that a character (1 byte) can only take the values from 0 to 255,<br>
we can just take 255 guesses at the value of `s[10]` to find the flag.<br>

```python
from z3 import *

s = [Int(f"s{i}") for i in range(34)]
known = "VishwaCTF{"
for i in range(len(known)):
    s[i] = ord(known[i])
s[33] = ord("}")

for i in range(256):
    s[10] = i

    solver = Solver()
    solver.add(s[3] + s[4] + s[1] + s[7] - s[8] * s[2] * s[6] * s[5] - s[11] - s[9] - s[10] == -52316790)
    solver.add(s[3] - s[4] - s[6] + s[9] + s[8] * s[11] * s[10] - s[2] + s[5] + s[7] * s[12] == 285707)
    solver.add(s[11] + s[10] * s[4] + s[3] - s[12] * s[7] - s[13] - s[5] * s[9] * s[6] + s[8] == -797145)
    solver.add(s[4] + s[12] - s[7] * s[11] - s[9] - s[5] * s[6] - s[14] - s[8] * s[13] * s[10] == -289275)
    solver.add(s[13] + s[14] + s[7] + s[6] - s[12] - s[15] * s[11] - s[5] + s[8] * s[10] * s[9] == 666868)
    solver.add(s[12] + s[15] * s[16] + s[11] + s[13] - s[10] + s[6] * s[8] - s[7] - s[9] + s[14] == 9837)
    solver.add(s[7] + s[11] - s[8] + s[16] * s[13] - s[17] - s[14] - s[9] + s[10] * s[15] - s[12] == 9858)
    solver.add(s[17] + s[12] + s[9] - s[18] - s[8] - s[15] + s[16] + s[11] * s[14] * s[13] - s[10] == 296504)
    solver.add(s[11] * s[13] * s[18] * s[16] - s[17] - s[10] + s[9] + s[15] * s[12] - s[19] - s[14] == 10963387)
    solver.add(s[17] + s[16] + s[20] + s[12] - s[14] * s[18] * s[15] * s[19] - s[13] - s[11] - s[10] == -65889660)
    solver.add(s[16] - s[19] - s[15] + s[11] * s[13] + s[18] + s[21] * s[12] + s[14] + s[17] * s[20] == 13340)
    solver.add(s[18] * s[16] + s[17] * s[15] - s[20] - s[12] - s[19] * s[14] + s[22] + s[13] * s[21] == 4641)
    solver.add(s[15] + s[20] + s[18] + s[21] + s[13] * s[19] - s[22] - s[16] - s[14] + s[17] * s[23] == 6428)
    solver.add(s[19] * s[24] + s[15] * s[20] + s[16] * s[14] + s[23] - s[18] * s[21] - s[22] * s[17] == 7851)
    solver.add(s[19] + s[24] + s[22] + s[21] + s[25] + s[16] + s[18] + s[20] * s[23] - s[15] + s[17] == 2997)
    solver.add(s[17] * s[23] + s[20] * s[25] - s[16] + s[26] * s[21] - s[24] + s[22] * s[19] * s[18] == 342425)
    solver.add(s[20] + s[26] + s[24] * s[17] + s[27] * s[22] * s[25] - s[21] - s[19] * s[18] + s[23] == 243251)
    solver.add(s[24] + s[22] + s[25] * s[21] - s[28] - s[19] - s[26] * s[27] * s[20] - s[23] + s[18] == -434772)
    solver.add(s[28] + s[19] + s[25] + s[29] - s[24] - s[21] - s[23] + s[27] - s[22] * s[26] + s[20] == -4957)
    solver.add(s[21] + s[30] + s[26] + s[22] * s[23] - s[29] + s[20] - s[24] * s[25] - s[27] - s[28] == -1625)
    solver.add(s[22] + s[26] + s[25] + s[30] + s[23] - s[24] - s[29] - s[31] - s[21] - s[27] - s[28] == -144)
    solver.add(s[29] + s[30] + s[31] - s[26] - s[25] - s[23] - s[28] - s[27] - s[22] - s[32] * s[24] == -7001)
    solver.add(s[33] + s[25] - s[31] * s[23] + s[27] - s[26] * s[32] + s[30] - s[24] * s[29] - s[28] == -18763)
    if (solver.check() == sat):
        print("FOUND")
        print(f"s[10] = {s[10]}")
        print(solver.model())
        break
```

Flag: `VishwaCTF{N3V3r_60NN4_61V3_Y0U_UP}`

<br>
<br>
<br>

## Phi-Calculator (136 Points, 322 Solves)

This challenge was much easier, with the source code provided in python.

```python
#============================================================================#
#============================Phi CALCULATOR===============================#
#============================================================================#

import hashlib
...
username_trial = "vishwaCTF"
bUsername_trial = b"vishwaCTF"

key_part_static1_trial = "VishwaCTF{m4k3_it_possibl3_"
key_part_dynamic1_trial = "xxxxxxxx"
key_part_static2_trial = "}"
key_full_template_trial = key_part_static1_trial + key_part_dynamic1_trial + key_part_static2_trial
...
def check_key(key, username_trial):

    global key_full_template_trial

    if len(key) != len(key_full_template_trial):
        return False
    else:
        # Check static base key part --v
        i = 0
        for c in key_part_static1_trial:
            if key[i] != c:
                return False

            i += 1

        # TODO : test performance on toolbox container
        # Check dynamic part --v
        if key[i] != hashlib.sha256(username_trial).hexdigest()[4]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[5]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[3]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[6]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[2]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[7]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[1]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[8]:
            return False

        return True
...
```

As seen from the code above, the check_key function checks our input directly with the hash of `username_trial`.
We can just find these values and use them as our key.

```python
>>> import hashlib

>>> bUsername_trial = b"vishwaCTF"

>>> key_part_static1_trial = "VishwaCTF{m4k3_it_possibl3_"
>>> key_part_dynamic1_trial = "xxxxxxxx"
>>> key_part_static2_trial = "}"
>>> key_full_template_trial = key_part_static1_trial + key_part_dynamic1_trial + key_part_static2_trial

>>> hashlib.sha256(bUsername_trial).hexdigest()[4]
'b'
>>> hashlib.sha256(bUsername_trial).hexdigest()[5]
'7'
>>> hashlib.sha256(bUsername_trial).hexdigest()[3]
'c'
>>> hashlib.sha256(bUsername_trial).hexdigest()[6]
'd'
>>> hashlib.sha256(bUsername_trial).hexdigest()[2]
'c'
>>> hashlib.sha256(bUsername_trial).hexdigest()[7]
'5'
>>> hashlib.sha256(bUsername_trial).hexdigest()[1]
'1'
>>> hashlib.sha256(bUsername_trial).hexdigest()[8]
'7'
>>> key_part_dynamic1_trial = "b7cdc517"
>>> key_full_template_trial = key_part_static1_trial + key_part_dynamic1_trial + key_part_static2_trial
>>> key_full_template_trial
'VishwaCTF{m4k3_it_possibl3_b7cdc517}'
```

Flag: `VishwaCTF{m4k3_it_possibl3_b7cdc517}`
