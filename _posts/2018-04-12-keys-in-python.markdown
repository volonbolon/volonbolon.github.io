---
layout: post
title: "Keys in Python"
date: 2018-04-12 19:05:39 GMT
tags: blockchain python bitcoin crypto
---

We [already saw](https://iamvolonbolon.tumblr.com/post/172725515985/blockchain-keys) the basic concepts behind the creation of keys and address creation. Let’s try to implement them. I’m going to use Python because it’s easier to follow. 

Following the [Standards for Efficient Cryptography](http://www.secg.org/sec2-v2.pdf), we define the constants. In the document, please look for “Recommended 256-bit Elliptic Curve Domain Parameters over Fp“ (secp256k1)

```
p = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFC2F
order = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141
# Remember the equation we are using is `y^2 % p == x^3 + 7
a = 0x0000000000000000000000000000000000000000000000000000000000000000
b = 0x0000000000000000000000000000000000000000000000000000000000000007
# This is the generator point defined in the standard
g = (0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798,
0x483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8)
```

For pedagogical reasons, I’ve chosen [python-ecdsa](https://github.com/warner/python-ecdsa). It’s a pure python library, really easy to understand. In production settings, it might be worth to explore C alternatives for speed, but more importantly,  this library should NOT be used in systems where `os.urandom()` is weak. Once installed the library, and imported to the Python unit, we can create a curve like this

```
# First, let's define the curve
elliptic_curve_secp256k1 = ecdsa.ellipticcurve.CurveFp(p, a, b)
# And, from the curve, extract the generator
generator_secp256k1 = ecdsa.ellipticcurve.Point(elliptic_curve_secp256k1, g[0], g[1], order)
# Now, we need to define the object identifiers for the elliptic curve domain parameters
certicom_arc_oid = (1, 3, 132, 0, 10)
#Finally, the curve
secp256k1 = ecdsa.curves.Curve("SECP256k1", elliptic_curve_secp256k1, generator_secp256k1, certicom_arc_oid)
```

As we know, the private key is only a random number in the range [0, 2^256], we can pick it with a simple python function. The key here is to grab as many entropy as possible for the random draft. the `os.urandom` function is described in the [documentation](https://docs.python.org/2/library/os.html) as capable of “Return a string of n random bytes suitable for cryptographic use”. The documentation also points to [SystemRandom](https://docs.python.org/2/library/random.html#random.SystemRandom) as an easy-to-use interface. 

```
def private_key():
	key = sum([(SystemRandom().randrange(2) * 2 ** idx) for idx in range(256)])
	return key
```

Once we have the secret key, we compute the public key by performing the multiplication. Remember, the public key is a point in the curve, in other words, it’s an ordered pair. 

```
public_key_point = secret_key * generator_secp256k1
```

When exposed, the public key is prefixed with `04`, and then, the value of the x and y coordinates expressed as hex numbers: 

```
04[hex(x)hex(y)]
```

But, as [discussed before](https://iamvolonbolon.tumblr.com/post/172841967580/blockchain-public-key-representations), since you can retrieve the value of `y` given `x` (and the sign of `y`), the compressed format is gaining popularity all the time. The sign of y is indicated by the prefixes `02` (positive ~ even) or 03 (negative ~ odd).

```
def produce_public_compressed_key_from_point(point):
	point_y_is_odd = point.y() & 1
	if point_y_is_odd:
		key = '03' + '%064x' % point.x()
	else:
		key = '02' + '%064x' % point.x()
	return key
``` 

[Gist](https://gist.github.com/volonbolon/ab707ecf2ccc97be19a71282ffeff461)