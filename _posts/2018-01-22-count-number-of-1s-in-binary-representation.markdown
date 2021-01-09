---
layout: post
title: "Count number of 1s in binary representation"
date: 2018-01-22 13:21:48 GMT
tags: cs challenges algorithm
---

Given an unsigned integer, we want to count how many `1` there are in its binary representation.

The naive approach would be to try to convert the given number (presented in base 10) to binary. But that seems preposterous, after all, the system already represents all integers as binary internally. 

We can inspect the bits with bitwise operators. In particular, we can use [AND](https://en.wikipedia.org/wiki/Bitwise_operation#AND) to extract the ones from the original number comparing it to a number of masks. 

```
0110 & 0001 = 0000
0110 & 0010 = 0010 
0110 & 0100 = 0100
0110 & 1000 = 0000
```

If the result of the comparison is not zero, we can increase the ones tally. 

```
extension UInt {
    func countOnesMultipleMasks() -> Int {
        let masks:[UInt] = [1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, 32768, 65536, 131072, 262144, 524288, 1048576, 2097152, 4194304, 8388608, 16777216, 33554432, 67108864, 134217728, 268435456, 536870912, 1073741824, 2147483648, 4294967296, 8589934592, 17179869184, 34359738368, 68719476736, 137438953472, 274877906944, 549755813888, 1099511627776, 2199023255552, 4398046511104, 8796093022208, 17592186044416, 35184372088832, 70368744177664, 140737488355328, 281474976710656, 562949953421312, 1125899906842624, 2251799813685248, 4503599627370496, 9007199254740992, 18014398509481984, 36028797018963968, 72057594037927936, 144115188075855872, 288230376151711744, 576460752303423488, 1152921504606846976, 2305843009213693952, 4611686018427387904, 9223372036854775808]
        var ones = 0
        for mask in masks {
            if mask & self != 0 {
                ones += 1
            }
        }
        return ones
    }
}
```

The array with hard coded values is distasteful, and we can easily produce it on the go. 

```
extension UInt {
    static var numberOfBits: UInt {
        let max = Double(UInt.max)
        let log = UInt(log2(max))
        let numberOfBits = log + 1
        return numberOfBits
    }

    static func createMasks() -> [UInt] {
        var masks: [UInt] = [1]
        let numberOfBits = UInt.numberOfBits
        for i in 0..<(numberOfBits-2) {
            masks.append(2 << i)
        }
        return masks
    }
}
```

And, since we are already manipulating bits to build the mask, we can forget entirely about the array, and just update the mask on each iteration. 

```
extension UInt {
    func countOnesWithMask() -> UInt {
        let numberOfBits = UInt.numberOfBits
        var mask:UInt = 1
        let upper = numberOfBits - 1
        var ones:UInt = 0
        for i in 0..<upper {
            if mask & self == mask {
                ones += 1
            }
            mask = 2 << i
        }
        return ones
    }
}
```

But even then, we are wasting a few cycles. Most numbers are going to be much smaller than the maximum possible number. Yet, we are checking up 2 ^ (number of bits). We can manipulate the input, moving from the most significant one found, until zero. At that point, the tally is complete. 

```
extension UInt {
    func countOnes() -> UInt {
        var n = self
        var ones:UInt = 0
        let mask:UInt = 1
        while n != 0 {
            let unmasked = mask & n
            if unmasked == 1 {
                ones += 1
            }
            n = n >> 1
        }
        return ones
    }
}
```
