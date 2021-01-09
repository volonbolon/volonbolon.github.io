---
layout: post
title: "Reverse words in a string"
date: 2018-01-03 14:27:16 GMT
tags: cs challenges algorithm
---

The challenge is to reverse the words in a string. In Swift, the `struct` offers limited access to the underlying `characters` array. That means, in order to operate at the character level, we need to get a buffer with it. 

```
let str = "Call me Ishmael."
let strArray = Array(str) // ["C", "a", "l", "l", " ", "m", "e", " ", "I", "s", "h", "m", "a", "e", "l", "."]
```

Now, we can use the array techniques to manipulate the string. For instance, we can reverse the content of the array with two pointers advancing towards each other from both ends. 

```
while down < up {
            (chars[down], chars[up]) = (chars[up], chars[down])

            down += 1
            up -= 1
        }
```

By swapping elements like this, the string is going to be completely reversed: `.leamhsI em llaC`. But the challenge is to reverse the order in which the words appear, not the letters. 

One more pass would do the trick. All we need to do is to find the tokens that separate the string into words (let’s say, the space character), and then reverse the characters within each group. 

So, next step is to retrieve the indices of all the spaces in the string. In the 70s *Donald Knuth* and *Vaughan Pratt* and independently, *James H. Morris*, conceived an algorithm to search for a substring. It makes little sense to use *Knuth–Morris–Pratt* here because the cardinality of substring is 1. *Knuth–Morris–Pratt* perform better when the substring to search is long enough. When we get a mismatch, the substring itself embodies sufficient information to determine where the next match could begin, thus bypassing re-examination of previously matched characters. But, nevertheless, let’s implement. 

```
private func indicesOf(subString: String, searchArray: [String.Element]) -> [Int] {
        // https://en.wikipedia.org/wiki/Knuth–Morris–Pratt_algorithm
        var indices: [Int] = []

        let word = Array(subString)

        var m = 0
        var i = 0
        while m + i < searchArray.count {
            if word[i] == searchArray[m + i] {
                if i == word.count - 1 {
                    indices.append(m)
                    m += i + 1
                    i = 0
                } else {
                    i += 1
                }
            } else {
                m += 1
                i = 0
            }
        }

        return indices
    }
```

Now, with the indices of the tokens at our disposal (`[8, 11]`), we can *split* the string, and reversed the letters within each word. 

```
    func reversedWords() -> String {
        var chars = Array(self)
        let range = Range(0..<(chars.count-1))
        self.reverseChars(chars: &chars, range: range)

        let indices = self.indicesOf(subString: " ", searchArray: chars)

        print(indices)
        var lower = 0
        for i in indices {
            let range = Range(lower..<(i-1))
            self.reverseChars(chars: &chars, range: range)
            lower = i+1
        }
        let lastRange = Range(((indices.last!)+1)..<(chars.count-1))
        self.reverseChars(chars: &chars, range: lastRange)
        return String(chars)
    }
```

And we are done: *Ishmael. me Call* 

As usual, [here](https://gist.github.com/volonbolon/025f1afef2a8117fd1f19511f639d61a) a playground with a full implementation. 