---
layout: post
title: "Integer / String conversion"
date: 2018-01-10 18:25:15 GMT
tags: cs challenges algorithm
---

The idea is to write a couple of helpers to convert a String to a Signed Integer, and back. 

Yes, I am aware you can cast values: 

```
let x: Int = 42

var asString = String(x)
assert("42" == asString)

let asInt = Int(asString)
assert(x == asInt)
```

But let’s try to explore alternative solutions. I will split this into two different posts to keep them concise. [String to Integer Conversion](/https://volonbolon.me/2018/01/10/string-to-integer-conversion.html) and [Integer to String Conversion](https://volonbolon.me/2018/01/10/integer-to-string-conversion.html)

As usual, a playground with examples is uploaded to [GitHubGist](https://gist.github.com/volonbolon/bf3dd3cb66dc8fd142c6ebfa273974cd)