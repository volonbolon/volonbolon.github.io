---
layout: post
title: "Coding Algebra"
date: 2018-03-05 12:15:29 GMT
tags: linear algebra math is fun
---

One pretty neat, and yet, little-known application of Linear Algebra is coding~encoding schemes that can detect errors in transmission on the spot.

### UPC
Perhaps, the most used code of these type is UPC. Introduced in 1973, and now ubiquitous, consists of 11 numeric digits and a 12th digit used to check the validity of the code.

![UPC Example](https://www.officialeancode.com/sample_codes/UPC%20Code%20sample.jpg “UPC”)


We can also think the code as a vector with 11 components in the interval [0…9] (from the set of Integers). To those 11 components, we add a 12th component (called the *check digit*) composed in such a way that, when we compute the dot product of that vector with a vector called *check vector*, the result, in mod 10, is zero. The check vector is `{3,1,3,1,3,1,3,1,3,1,3,1}`.

For instance, let’s consider the code `19019807207` (which happens to be the UPC for an iPhone 7). We need to compute the *check digit*. If we call the *check vector* `v`, the code vector `c` and the check digit `d`, we know that, after rearranging the components, we have

```
3 * (v1 + v3 + v5 + v7 + v9 + v11) + (v2 + v4 + v6 + v8 + v10) + d = 0 mod 10
```

In our example we have

```
3 * (1 + 0 + 9 + 0 + 2 + 7) + (9 + 1 + 8 + 7 + 0) + d
57 + 25 + d = 82 + d
```

`d` has to be 8.

UPC will detect all single errors and most transposition errors for adjacent components, for instance, if the scanner reads the code as `{190198702078}`, the result of `c.v` is going to be `4` in mod 10. Something went wrong.

### ISBN-10
In 1970 [ISO](https://en.wikipedia.org/wiki/International_Organization_for_Standardization) published ISO 2108, standardizing codes for books.

This time, the code vector is composed of 9 components in the interval [0…10], encoding information such as country of origin, publisher, and book id. To those, we append the *check digit*, but this time the rules to compute it at a little different. For starters, the `check vector` is `{10, 9, 8, 7, 6, 5, 4, 3, 2, 1}`, and the result of `v.c` should be zero, but in mod 11. Let’s consider `{9, 5, 0, 7, 4, 2, 4, 5, 6}` [^1]

```
9 * 10 + 5 * 9 + 0 * 8 + 7 * 7 + 4 * 6 + 2 * 5 + 4 * 4 + 5 * 3 + 6 * 2 + d
261 + d 
```

In mod 11, 261 is 8, the *check digit* should be 3 then.

ISBN-10 has been designed to detect all adjacent transposition errors.

### ISBN-13
To gain compatibility with [EAN-13](https://en.wikipedia.org/wiki/International_Article_Number#EAN-13_encoding), ISBN has been extended to 13 digits. The *check vector* has the same structure of the check vector in UPC, and the dot product should be zero in mod 13.

Let’s try `{9, 7, 8, 9, 8, 7, 4, 5, 3, 5, 3, 9}` [^2]

```
3 * (v1 + v3 + v5 + v7 + v9 + v11) + (v2 + v4 + v6 + v8 + v10 + v12) + d
3 * (9 + 8 + 8 + 4 + 3 + 3) + (7 + 9 + 7 + 5 + 5 + 9) + d
147 + d
```

147 mod 13 is 4, so `d` has to be `9`.

ISBN-13 improves on ISBN-10 and is also capable to detect all single errors and adjacent transposition.

### Codabar System
Pretty much the same scheme is used to validate credit card numbers. 

The number is composed of 15 digits, assigned by the Bank issuing the card, and a *check digit*. 

Most banks use a system called Codabar to compute the check digit. Because Codabar is self-checking, most standards do not require the check digit, that’s not the case with [Luhn algorithm](https://en.wikipedia.org/wiki/Luhn_algorithm), the one actually used in credit cards. We are going to use a vector with 16 components from the Integer interval `[0…9]`, and now, we have to add the number of odd components greater than 4. We will call this tally `h`. 

```
c.v + h = 0 (mod 10)
```

With the *code vector* `{5, 6, 1, 0, 5, 9, 1, 0, 8, 1, 0, 1, 8, 2, 5, d}` we compute the dot product. Note that the check vector is pretty much the one used in UPC, except the odd component is `2` and not `3`. In this case `h = 5`

```
{5, 6, 1, 0, 5, 9, 1, 0, 8, 1, 0, 1, 8, 2, 5, d}.{2, 1, 2, 1, 2, 1, 2, 1, 2, 1, 2, 1, 2, 1, 2, 1} + 5
2 * (5 + 1 + 5 + 1 + 8 + 0 + 8 + 5) + (6 + 0 + 9 + 0 + 1 + 1 + 2 + d) = 85 + d + 5
90 + d = 0 (mod 10)
```

Now we have our *check digit*, `0`[^3]. 

[^1]: [Adan Buenosayres](https://www.iberlibro.com/9789507424564/Adan-Buenosayres-Leopoldo-Marechal-9507424563/plp)
[^2]: [Psicopyme](http://editoresasociados.com.ar/catalogo/psicopyme/)
[^3]: [Test Credit Card Account Numbers](https://www.paypalobjects.com/en_AU/vhelp/paypalmanager_help/credit_card_numbers.htm)