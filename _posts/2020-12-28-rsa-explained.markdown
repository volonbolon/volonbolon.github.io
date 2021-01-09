---
layout: post
title: "RSA explained"
date: 2020-12-28 21:46:10 GMT
tags: crypto math is fun mathematics
---

One of the oldest asymmetric cryptosystems, develop in the 70s and still used today. The story goes that, one night in 1977, [Ron Rivest](https://en.wikipedia.org/wiki/Ron_Rivest) drank a few glasses of wine, and, unable to sleep, found a trap door function that was alluding him, and his partners [Adi Shamir](https://en.wikipedia.org/wiki/Adi_Shamir) and [Leonard Adleman](https://en.wikipedia.org/wiki/Leonard_Adleman). 
It is a great achievement of 20th-century math, and yet, technically, is not really hard to follow. Let's try it.

## Key Generation
First, we need to choose two prime numbers. In a real-world scenario, we would be using huge prime numbers,  hundreds of digits long, but as an illustration, let's pick two small primes, 5 and 13. 

```
p = 5, q = 13
```

Now, we compute the product of these two numbers

```
modulus = 65
```

The modulus length, usually expressed in bits, is the key length. It is distributed as part of the public key. 

Now, we compute [Carmichael function](the https://en.wikipedia.org/wiki/Carmichael_function). Which is, basically, the number of caprimes with `modulus` in the range `1 < n < modulus`. Instead of just listing all the caprimes with modulus, and and since `p` and `q` are prime, `λ(p) = φ(p) = p − 1` and likewise `λ(q) = q − 1`. Hence `λ(modulus) = lcm(p − 1, q − 1)`. 

In our case, the least common multiple, and thus, our λ for our modulus is 65. 

The lambda is kept private. 

Now, we need to pick an integer, which we are going to call `e` such that `1 < e < λ(modulus)` and is coprime with `λ(modulus)

We can use Python to produce a list of candidates: 

```
>>> from math import gcd
>>> def coprime(a, b):
...   return cgd(a, b) == 1
...
>>> p = 5
>>> q = 13
>>> phi = (p-1) * (q-1)
>>> e_candidates = [e for e in range(2, phi) if coprime(e, phi)]
[5, 7, 11, 13, 17, 19, 23, 25, 29, 31, 35, 37, 41, 43, 47]
```
We can produce any number from that list as `e`, which, along with `modulus` is part of the public key. 

Finally, we need to produce our private key, an integer (`d`) that should solve: 

```
d * e = 1 (mod λ(modulus))
```

If, for instance, in our example, we pick 35 as our public `e`, then we can generate the following list of candidates for `d`

```
[11, 59, 107, 155, 203, 251, 299, 347, 395, 443, 491, 539, 587, 635, 683, 731, 779, 827, 875, 923, 971]
``` 
We can pick any `d` from the list, and we should keep it to ourselves. 

## Key Distribution
The beauty of the system is that the key can be distributed through a reliable, although not necessarily a secret channel. 

The public keys will allow anyone to encrypt messages that will be only readable for the one holding the private key. It would be unfeasible to either guess or derive the private key from the public key.

## Encryption / Decryption
If Alice wants to share with me the "Answer to the Ultimate Question of Life, the Universe, and Everything", but doesn't want Charlie to know, I will need her to know my public key, composed from e and the modulus

```
>>> from collections import namedtuple
>>> public_struct = namedtuple("public", "e mod")
>>> public = public_struct(35, 65)
>>> public
public(e=35, mod=65)
```

Then she will be able to encrypt the message, by multiplying itself as many times as indicated by `e`, mod `modulus`. 

```
>>> t = 42
>>> c = (t ** public.e) % public.mod
>>> c
48
```

`48` is our cipher text, Charlie has no way to know anything about the "answer to the ultimate question of life", only me, holding the private key 

```
>>> private_struct = namedtuple("private", "d mod")
>>> private = private_struct(587, 65)
```

Would be able to decrypt it: 

```
>>> decrypted = (c ** private.d) % private.mod
>>> decrypted
42
```