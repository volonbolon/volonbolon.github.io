# Swift Generators
---
layout: post
title: "Swift Generators"
date: 2021-02-10 22:54:38 GMT
tags: swift design_patterns
description:
---

Generators functions allow you to declare a function that can receive multiple inputs, and produce just in time the output. Because the generation of values is performed lazily, we don't need to allocate a large number of resources to build up complex sequences in advance. 

For instance, let's say we want to produce the Fibonacci sequence, up to the 50th element. We know that's expensive, and we might want to introduce memoization. That would certainly be a smart move if we need to compute the sequence more than once. 

```
var fibCache: [Int: Int] = [:]

func fibonacci(_ number: Int) -> Int {
    if let fib = fibCache[number] {
        return fib
    }

    let newValue = number < 2 ? number : fibonacci(number - 2) + fibonacci(number - 1)
    fibCache[number] = newValue 
    return newValue
}

for i in 1...50 {
    print(fibonacci(i))
}
```

But then, we are incurring the cost of holding a vast cache to speed up computation. 

A different approach would be to compute values only when needed. Following the Fibonacci computation, we can reduce the data stored to just two variables. All we need to do is to adhere to `IteratorProtocol`, which requires the definition of `func next() -> Self.Element?`

```
struct FibonacciIterator: IteratorProtocol {
    private var a = 0
    private var b = 0

    init() {
        a = 0
        b = 0
    }

    mutating func next() -> Int? {
        let (f, o) = a.addingReportingOverflow(b) // the returned tuple will let us know if we are overflowing Int
        guard !o else {
            return nil
        }

        if a == 0 { // special case, first iteration
            a = 1
        } else {
            a = b
            b = f
        }

        return f
    }
}
```

Because `IteratorProtocol` is tightly linked with the `Sequence` protocol, we can create our own Sequence, and by exposing an iterator, for loops are a breeze.

```
struct Fibonacci: Sequence {
    func makeIterator() -> some IteratorProtocol {
        FibonacciIterator()
    }
}

var fi = Fibonacci()

for f in fi {
    print(f)
}
```