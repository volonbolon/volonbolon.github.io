---
layout: post
title: "Why 0.1 + 0.2 == 0.30000000000000004"
date: 2021-01-26 18:11:34 GMT
tags: blog computer_science algorithm
description:
---

Everything in the computer memory is binary, including decimals numbers. 

A naive approach to store them would be to agree upon how many bits would be devoted to the integer part, and how many to the fractional part. Let's say we have a byte, and we agree that the first 4 bits would be used to encode the integer part, and the rest for the fractional part. Arithmetic would be a breeze, at the expense of range. Even if we devote more bits, let's say 24 bits for the integer part, we would be able to accommodate almost 16 million values, not a lot. 

A different and smarter approach would use a binary version of the scientific notation, splitting the number to be stored in a mantissa (the significant digits of the number) and an exponent. Here we also have a trade-off between precision and range. The more bits devoted to the mantissa, the better the precision, at the cost of a smaller exponent, or range. There are a vast array of different schemes, but the most popular is the one standardized by the Institute of Electrical and Electronics Engineers (IEEE). 

## IEEE 754 Standard 
It wasn't until the 80s that the IEEE presented its standard for floating-point arithmetics. Today it has been adopted by most of today's architectures. 

Its basic layout requires 32 bit (single precision), although there are layouts for 16 (half precision), 64 bit (double) and even 128 (quadruple precision)

| Size | Exponent | Mantissa|
| :--: | :-------: | :---: |
| 16   | 5        | 10      |
| 32   | 8        | 23      |
| 64   | 11       | 52      |
| 128   | 15       | 112      |


The leftmost bit is always the sign bit, followed by the exponent, and then the mantissa. Because all possible values that we would like to represent, apart from zero, will start with `1`, we assume is there, tacitly. Thus, for a single-precision mantissa, we have 24 bits as storage (the implicit 1 as the most significant value, and 23 explicit bits). 

### Conversion
To convert any decimal number (within the range) the first step is trivial, to tell if we are dealing with a positive or negative number. If positive, the sign bit would be zero, if negative 1. For example, if we want to compute the IEEE 754 representation of 26.79238, we need to start by asking ourselves if it is positive or negative. Since it is positive, our sign value is going to be `0`. 

Then, we need to convert the number into its binary representation, using a fixed point format. At this stage the sign of the value is not important, we already dealt with that. We can treat the value as positive. 

To get the integer part representation, we can keep halving it until we reach zero, the remainders of each division, in reverse order, are our bit pattern. For our example, 26 turns into `11010`

```
def int_to_binary(n):
    bits = []
    h = n // 2
    while n >= 1:
        bits.append(n - (h * 2))
        n = h
        h = n // 2

    # we need to reverse the calculated bits, 
    # and since we are here, let's produce a number
    b = 0
    for n in range(len(bits), 0, -1):
        b = b * 10 + bits[n-1]
        
    return b
```

For the decimal part, we should keep doubling it until the desire precision. We start by doubling '0.79238', to get '1.58476', we extract the 1, to get our first bit, and we keep going by multiplying '0.58476 * 2 = 1.16952', to get a second 1 for our bit pattern. The third bit is going to be a 0, because '0.16952  × 2 = 0.33904'. 

```
def dec_to_binary(n, prec):
    bits = ""
    while prec:
        n *= 2
        fb = int(n)
        bits += str(fb)
        if fb == 1:
            n -= fb
        prec -= 1
        
    return bits
```

Finally, we need to normalize the fixed-point representation, the decimal point is moved as many positions as needed to have only one non zero to the left of it. 

In our example, 26.79238, became the fixed-point representation 11010.1100101011011001, and we normalize it to 1.10101100101011011001 * 2 ^ 4 (the exponent is 4 because we move the decimal point 4 positions). 

The exponent calculated previously is unbiased, but the standards require a biased exponent. That way, we can input the sign of the exponent (negative exponents are used to fractional values), but leaving more room for positive exponents (which would be used more frequently). 

In order to illustrate the previous paragraph, let's consider the nonexistent case where the exponent is composed of 4 bits. We have 16 values. The underlying principles are exactly the same, it is only simpler to think about fewer bits.

If we choose to indicate the sign of our exponent using two's complement (a 1 in our left-most bit indicates a negative value), we still have 16 values, [-8, 7] with an equal number of positives and negatives (if we take 0 as a positive). 

But splitting our range in half might not be the best approach. We can tailor the range of negative and positive values as we want. Instead of using the actual binary value, we can subtract a given number. For instance, if we want a larger range for the positive exponent, we can bias it by -3, 0b0000 would become the representation of -3, and we can reach 12 as our maximum exponent (instead of 8 as in two's complement) while retaining some capacity for negative exponents (instead of the raw binary representation). 

| Binary | Decimal | Two's Complement | Biased (-3) | Biased (10)  |
| :--: | :--: | :--: | :--: | :--:  |
| 0b0000 | 0 | 0 | -3 | -10  |
| 0b0001 | 1 | 1 | -2 | -9  |
| 0b0010 | 2 | 2 | -1 | -8  |
| 0b0011 | 3 | 3 | 0 | -7  |
| 0b0100 | 4 | 4 | 1 | -5  |
| 0b0101 | 5 | 5 | 2 | -4  |
| 0b0110 | 6 | 6 | 3 | -3  |
| 0b0111 | 7 | 7 | 4 | -2  |
| 0b1000 | 8 | -8 | 5 | -1  |
| 0b1001 | 9 | -7 | 6 | 0  |
| 0b1010 | 10 | -6 | 7 | 1  |
| 0b1011 | 11 | -5 | 8 | 2  |
| 0b1100 | 12 | -4 | 9 | 3  |
| 0b1101 | 13 | -3 | 10 | 4  |
| 0b1110 | 14 | -2 | 11 | 5  |
| 0b1111 | 15 | -1 | 12 | 6  |

IEEE 754 slightly favors positive exponents with an offset equal to 2^(exponent size - 1) - 1. For a single-precision floating the bias is 127. That means that we need to add 127 to our exponent to get the biased value. In our example, 4 turns into 131, or 0b10000011. Now, all we need to do is to remove the leading 1 from the mantissa (remember, this is tacitly incorporated), and we have our IEEE 754 single-precision floating-point representation

| Decimal | Sign | Exponent | Mantissa | 
| :--: | :--: | :--: | :--: |
| 26.79238 | 0 | 10000011 | 10101100101011011001000 |

That being said, there is one more really important aspect of the biased exponent. The binary encodings are presented in lexical order. In two's complement 0b1000 (-8) is less than 0b0011 (3), but that not the case in biased exponents. 

## So why?
Armed with all the details, we now understand that floating-point are approximations to the decimal value. In particular, in binary, the only prime factor is 2, so you can only cleanly express fractions whose denominator has only 2 as a prime factor. In binary, 1/2, 1/4, 1/8 would all be expressed cleanly as decimals, while 1/5 or 1/10 would be repeating decimals.

Let's calculate the representations of 0.1 and 0.2 

| Decimal | Sign | Exponent | Mantissa | Binary Scientific |
| :--: | :--: | :--: | :--: | :--: |
| 0.1 | 0 | 01111011 | 10011001100110011001100 | 1.10011001100110011001100 * 2 ^ -4
| 0.2 | 0 | 01111100 | 10011001100110011001100 | 1.10011001100110011001100 * 2 ^ -3 |

We can see the pattern of bits (*1100*). When you perform math on these repeating decimals, you end up with leftovers which carry over when you convert the computer’s base-2 (binary) number into a more human-readable base-10 representation.

In order to sum these two, we will need to convert from IEEEE 754 representation to binary scientific. Both numbers are positive, for 0,1 the exponent gets translated to -4, and for 0.2 to -3

Then, we need to match the exponents, moving the decimal point and adjusting the values. 

```
0.11001100110011001100110 * 2 ^ -3
1.10011001100110011001100 * 2 ^ -3
----------------------------------
10.01100110011001100110010 * 2 ^ -3 == 1.001100110011001100110010 * 2 ^ -2
```

When converted to decimal, we get close to 0.3, but no 0.3