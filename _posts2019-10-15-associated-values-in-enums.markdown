---
layout: post
title: "Associated values in Enums"
date: 2019-10-15 15:34:35 GMT
categories: Swift
---

In Swift, Enumerations are much more flexible than in other languages like C. We Can define them not just as a set of `integers`, but as anything, provided we gave each case a raw value. 

More importantly, cases can specify *associated values* of any type to be stored along with each value, much as unions or variants in other languages. 

Swift enumerations can store associated values of any given type, including other enums. The values can be different for each case in the enumeration. 

```
enum Support: String {
    case vynil = "Vynil"
    case mp3 = "MP3"
    case cd = "Audio CD"
}

enum Product {
    case book(String, String, Double, Int, Int)
    case album(String, String, Double, Int, Support)
}

let theIliad: Product = .book("The Iliad", "Homer", 14.99, -1184, 190)
let blackstar: Product = .album("David Bowie", "Blackstar", 22.34, 2016, .vynil)

let products = [theIliad, blackstar]

products.forEach { (product: Product) in
    switch product {
    case .book(let title, let author, let price, let year, let pages):
        print("\(title) by \(author) was published in \(year). It cost $\(price), and contains \(pages)")
    case .album(let artist, let name, let price, let year, let support):
        print("\(artist) published \(name) in \(year). It cost $\(price) and it is distributed as \(support.rawValue)")
    }
}

/*
 The Iliad by Homer was published in -1184. It cost $14.99 and contains 190
 David Bowie published Blackstar in 2016. It cost $22.34 and it is distributed as Vynil
 */
```

## You have to choose
Unfortunately, in Swift enums, we can either have Associated Values or Raw Values, not both. The raw value of an enumeration is formalized with the [RawRepresentable](https://developer.apple.com/documentation/swift/rawrepresentable) protocol which states:

> With a RawRepresentable type, you can switch back and forth between a custom type and an associated RawValue type without losing the value of the original RawRepresentable type.

With a raw value, we are guaranteed to be able to reconstruct the original case. 

```
let support: Support = .vynil
let vynilRawValue = support.rawValue // "Vynil"
let reconstructedSupport = Support(rawValue: vynilRawValue) // .vynil
```

In contrast, with associated values, we cannot reconstruct the original value from its raw value, because there is no way to tell which associated value to choose, breaking the promise of `RawRepresentable`. 