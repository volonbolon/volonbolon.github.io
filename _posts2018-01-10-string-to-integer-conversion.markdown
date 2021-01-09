---
layout: post
title: "String to Integer Conversion"
date: 2018-01-10 18:18:57 GMT
categories: cs challenges algorithm
---

The input would be something like `”42”`, or perhaps `”-42”`. There are at least three things to consider. 

1. To correctly identify negative numbers
2. To parse each character as an integer
3. To construct the actual number value given the integers parsed in 2. 

To solve 1. all we need to do is to inspect the first character of the string, looking for the special case `”-”`. If we find it, we can raise a flag to remember to convert the number constructed in 3. to its negative. 

The solution for 2 is quite interesting, and remind me one of the [best books](https://www.amazon.com/dp/0131103628/) about programming I’ve ever read. I’ve never written a “hello world!” program. And as far as I can remember, the first one I wrote in C was one to print a table of ASCII codes, and their character representation. Why am I digressing like this? Because there you have a way to translate a character to an integer. And, more importantly, in ASCII, the characters representing numbers are all placed together and sorted. In other words, if you happen to know the value of `”0”`, then you know that the value of `”1”` is the next integer. It doesn’t matter the actual value of `”0”`, the difference `”1” - “0” == 1` 

So far, we managed to translate the string into an array of integers. Now we need to think how to convert the array into an actual number. From grade school, we should remember that the digits of a number tell us a lot from its own value, but tell us even more with its position. The first `2` in `22` is telling us something different than the second one. In particular, given its position, it is telling us that we need to multiply it by `10`. In other words, by taking into consideration the position of the digit, we can construct the actual number. 

Perhaps the easier way to compute the actual number value would be to go from right to left, as we’ve done in grade school. We can start at the unit column, and as we progress to the left, we should multiply the value found there by an order of magnitude higher. 

But there are a couple of issues here. The first one is that we will need to keep track of the position of each value, and then, we need to perform two operations even before starting the sum. We will need to take the exponent of 10, and then we will need to multiply that to the value in the position. 

What if, instead of tracking the position, and calculate the exponent in each iteration, we leave that to the actual structure of the buffer. Let’s say we have `[1,6,0,2]`. If we start looping at the left, we just read 1, make it the value of our number, and move to the next step. Since we are moving to the next position, we multiply our number by `10`, and then add to it the number we are reading `6`. So far, we have `16`. One more lap, and we multiply the number by `10`, converting it to `160`, and add the read digit. One last loop. The number is converted to `1600` by multiplying it by `10`, and by adding `2`  we finally get the expected value `1602`. And all of that without tracking the position, and saving one operation per loop. The optimization is pretty popular, useful in checksums, and based on the [Horner’s Scheme](https://en.wikipedia.org/wiki/Horner%27s_method). 

For historical reasons, I am converting Unicode to Ascii

```
extension Character {
    var asciiValue: Int {
        get {
            let s = String(self).unicodeScalars
            return Int(s[s.startIndex].value)
        }
    }
}

extension String {
    var asInt: Int {
        let base = Array("0").first!.asciiValue
        var result = 0
        let negative = self.hasPrefix("-")
        let startIndex = negative ? 1 : 0
        let stringAsArray = Array(self)
        for index in startIndex..<stringAsArray.count {
            let c = stringAsArray[index]
            let delta = c.asciiValue - base
            result *= 10 // Horner's Rule http://mathworld.wolfram.com/HornersRule.html
            result += delta
        }
        if negative {
            result *= -1
        }
        return result
    }
}
```