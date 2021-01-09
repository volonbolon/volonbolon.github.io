---
layout: post
title: "Reverse an array of integers"
date: 2017-12-07 15:03:00 GMT
tags: cs challenges small things
---

We have an array composed by `ints`, `[96, 49, 70, 47, 24, 46, 60, 29, 49, 12]`, and we need to reverse it. 

That’s pretty easy. We just need to counters, one initially set to zero, the other to the ordinal of the last element in the array, and traverse the array from both ends, increasing the first and reducing the second counter while the second is still greater than the first. 

The interesting part is to try to swap the two integers in place and with no temp storage. 

A first approach would be to use simple arithmetics. Let’s say we want to swap 3 and 5. 

```
var x = 3
var y = 5

x = x + y // x becoms 8
y = x - y // y becomes 3
x = x - y // x is reset to 5
``` 

Multiplication and division and also be used. Basically, all we need is a symmetrical transformation. 

```
var x = 3
var y = 5

x = x * y // x becomes 15
y = x / y // y becomes 3
x = x / y // x is reset to 5
```

But there are two problems. The most important, is the risk of overflowing the type. Let’s say for the sake of brevity that we are working on an hypothetical unsigned type of 3 bits, and we want to swap 6 and 4. The sum of these two would be 2 (6 + 4 mod 8), instead of the expected 10. 

The other problem is specific to multiplication and division. Everything just fell apart if one of the ints is zero. 

We can avoid these two issues with the bitwise operator *Exclusive or (`xor`)* Xor will not generate a new int, it will just modified the underlying pattern of bits. 6 (`0b110`) ^ 4 (`0b100`) would become `0b010`. We are just shifting the bits in the binary representation, we are not performing an arithmetic operation. 

```
var l:[Int] = []
for _ in 0..<10 {
    let n = Int(arc4random_uniform(101))
    l.append(n)
}
print(l) // [18, 17, 16, 53, 82, 91, 70, 88, 58, 76]

var i = 0
var j = l.count - 1
while i < j {
    l[i] = l[i] ^ l[j]
    l[j] = l[i] ^ l[j]
    l[i] = l[i] ^ l[j]
    i += 1
    j -= 1
}
print(l) // [76, 58, 88, 70, 91, 82, 53, 16, 17, 18]
```