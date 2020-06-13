---
title: 'HackPack CTF 2020: Hiding in Plaintext'
date: 2020-05-18
categories: [writeups]
author: None
---

> Connect to CryptoKid's Mall of MirrXors at cha.hackpack.club:41715 to try
> your hand at guessing the flag!
>
> (Mr. Vernam says CryptoKid stole his ideas, but Miss Venona says he stole
> them...badly...)

<!--more-->

On connecting to the service we see the following, where the `a`s are our
input.

```
Welcome to CryptoKid's Hall of MirrXors!

Here is BASE64(ENC(FLAG, SESSION_KEY)) => /4aJAQQnEuQQ+BpN8jBXpXx0eGISE3Rcl0uHh5DH4gXUFis9UrPrU225W5CNK2/YKynOA6oGd7n0OA==

Wanna guess what FLAG is? aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa


Here is BASE64(ENC(GUESS, SESSION_KEY)) => +IuJBx4oQ/Fc7RMfvjwGt2k4KG4DACZOhRuQ1dzW7ySEGT5vS6anRHm0VNyOP3qUPSCfT6gnZOvmJPiL

How'd you do??

Hahahahahaha, try again...
```

In the description it is hinted that the encryption is xor. If that is the
case, we have `enc_flag = key ^ flag` and `enc_guess = key ^ guess`, but since
xor is self-inverse the flag can be reconstructed by computing `key = enc_guess
^ guess` and `flag = enc_flag ^ key`. We can write a small Python script

```python
from base64 import b64decode
from operator import xor

enc_flag = b64decode('/4aJAQQnEuQQ+BpN8jBXpXx0eGISE3Rcl0uHh5DH4gXUFis9UrPrU225W5CNK2/YKynOA6oGd7n0OA==')
guess = 60 * b'a'
enc_guess = b64decode('+IuJBx4oQ/Fc7RMfvjwGt2k4KG4DACZOhRuQ1dzW7ySEGT5vS6anRHm0VNyOP3qUPSCfT6gnZOvmJPiL')
print(bytes(map(xor, enc_flag, map(xor, enc_guess, guess))).decode())
```

to get the flag:

```
flag{n0t-th3-m0st-1mpr3ss1v3-pl@1nt3xt-vuln-but-wh0-c@r3s}
```
