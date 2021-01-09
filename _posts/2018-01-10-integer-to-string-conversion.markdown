---
layout: post
title: "Integer to String Conversion"
date: 2018-01-10 18:21:53 GMT
tags: cs challenges algorithm
---

It turns out, that here, we also have three basic problems, related to the ones we found in [String to Integer Conversion](https://iamvolonbolon.tumblr.com/post/169548302500/string-to-integer-conversion). 

1. Recognize if we are dealing with a negative number. 
2. Extract the digits of the original number
3. Translate each digit into its related character. 

To recognize if we are dealing with a negative number is trivial. All we need to do is to check its actual value. 

And to translate each digit into a character is also straightforward. We just need to remember that in ASCII (and Unicode UTF8, for instance), characters representing integers are in a contiguous space. If we know how to get to `”0”`, we know how to get to the rest. 

Which leave us thinking about how to solve 2. It is tempting to think about extracting the digits, and it would be nice, but, unfortunately, even when on screen we might be seeing `42`, in reality, integers are represented as binary, ie, `101010`. We cannot extract decimal digits from a binary. We need to compute them. 

One way to calculate the last digit of any number is to take the modulo of the number and 10. Let’s say we have 1602: `1602 % 10 = 2`. How to get the second integer (from right to left)? Let’s divide the input value by 10 on the realm of integers. We are down to 160, and if we get the modulo of 160, we end up with the second digit (0 in this case). 

The algorithm is emerging. We take the modulo of the value and 10 to get the digit, and then, we integer divide the value by then before the next lap. Each time, we save the digit into a character buffer. And we keep looping until the value is zero. And, if the `negative` flag has been raised, we just append the a `-` to the data store at the end. 

In the extract digits portion of the algorithm, special consideration needs to be taken with the zero. Not when obtained as a digit, but when given as a value. Remember we are using it as the stop condition of the loop. 

But alas, if we append items into an array like this, we will end up with the values in reversed order (we are reading from right to left). Obviously, we can use a stack to save characters, instead of an array. Or, even easier, we can reverse the array to recover the digits in the correct order. 

So far, we managed to deal with the negative values and to recover the digits. We need to convert them to characters. Well, once we get the digit, we just need to add it to the value of `”0”`, and save the corresponding character. 


```
extension Int {
    var asString: String {
        guard self != 0 else {
            return "0"
        }

        var chars: [Character] = []
        let base = Array("0").first!.asciiValue

        var value = self > 0 ? self : -self // We need to convert negatives to positive

        while value != 0 {
            let ascii = (value % 10) + base
            if let unicodeScalar = UnicodeScalar(ascii) {
                let c = Character(unicodeScalar)
                chars.append(c)
            }
            value /= 10
        }

        if self < 0 {
            let neg = Character("-")
            chars.append(neg)
        }

        // reverse buffer
        chars.reverse()

        let result = String(chars)
        return result
    }
}
```