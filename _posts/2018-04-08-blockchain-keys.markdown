---
layout: post
title: "Blockchain Keys"
date: 2018-04-08 15:40:26 GMT
tags: blockchain ecc crypto bitcoin
---

In the blockchain, there is nothing encrypted, but still, asymmetric cryptography is heavily used. Each wallet contains at least a pair of keys, a private one used to sign transactions (and to claim ownership of UTXO), and a public key, generated from the private one, and used to receive funds. 

Just the holder of the private key can produce the numerical signature of a transaction from its fingerprint. But anyone can verify the signature with the public key and the fingerprint. 

## Private Key
A blockchain private key is nothing but a random number in the range  `[0, 2^256]`. And here, random is paramount. Basic pseudo random generators are not enough. Remember, this random number is the only required proof to claim ownership (and to spend) associated funds. Obviously, [Hardware Random Number Generator](https://en.wikipedia.org/wiki/Hardware_random_number_generator) where entropy is obtained on some physical phenomena are better than [Pseudo-random Number Generator](https://en.wikipedia.org/wiki/Pseudorandom_number_generator), but even them might be good enough. They key here is that the generated number should be unpredictable[^1]. 

## Elliptic Curve
An elliptic curve is a [plane algebraic curve](https://en.wikipedia.org/wiki/Algebraic_curve). In other words, is the set of points on the Euclidean plane whose coordinates are zeros of a polynomial in two variables.  

The standard used in bitcoin is `secp256k1`, and choose a particular polynomial, `y2 = x3 + 7`. When evaluated in the real plane, you get a smooth curve symmetrical with respect to the x-axis. However, `secp256k1` is defined over the field of prime order *p*[^2]. That means you can kiss your smooth curve goodbye, now you are dealing with what seems to be a pattern of dots scattered through two dimensions. But that’s just a matter of representation, remember the definition, we just need a set. The mathematical properties we can find are associated with the object and not with the representation. 

For a pedagogical purpose, let’s reduce the prime order of the field to 11. We end up with a pattern like this one. 

![Primer Order 11](https://i.imgur.com/R9HT0nh.png)

### Addition
One of the properties of the elliptic curve is that if we draw a line between two points in the curve, the line will intersect the curve in exactly an additional point. This operation is called *addition*. If P1 and P2 have the same x coordinate, and different y coordinate, then the line joining them will be parallel to the y axis, and P3 is defined as a *point at infinity*. Now, if either P1 or P2 are defined as points at infinity, then we have P1+P2=P2 (iff P1 is the point at infinity), or P1+P2=P1 (when P2 is a point at infinity). That’s the reason a point at infinity is sometimes called *zero* over additions. 

### Multiplication
Now that we have defined *addition*, we can extend the definition to *multiplication* pretty easily. From classic arithmetic, we know that to multiple a value V by K means to add V to itself K times. Then, let’s take a point P on the elliptic curve, and add P to itself K times and we have ‘k*P’. 

To visualize the process, we can go back to the smooth representation of the curve over the real plane. To add a point to itself is equivalent to draw a tangent line to that point, find where the tangent intersects the curve, and then, reflect that point over the x-axis. The reflection point is the result of the multiplication. By performing the operation once, we will be adding P1 to itself, meaning, the result is equivalent to 2*P. 

Back to our field of prime order 11, we can multiply (6,5) * 2, and the result is going to be (3, 1)

## Public Key
We have a number chosen randomly in the range [0, 2^256], and we have defined such number our *private key*. We have also defined the multiplication operation over elliptic curves. And the standard defines a point in the elliptic curve called a *Generator Point*. We can then compute 

```
K = k * G
```

Where k is the the private key, G is the generator point, and K is defined as the Public Key. 

Because G is provided by the standard, given k, will always result in the same value for K, and the computation is pretty straightforward. But to find k given K and G is unfeasible (basically, you have to try every possible value, which on a field so vast as the one defined by the standard is unfeasible). Thus, we have defined a function with a wonderful property, is fairly easy to calculate in one direction, and almost impossible in the other one. A *trap door*. 

[^1]: Unpredictability is the property found in systems that prevent attackers to guess its output, even knowing previous results. 
[^2]: p = 2^256-2^32-2^9-2^7-2^6-2^4-1 = 1.15792089×10^77