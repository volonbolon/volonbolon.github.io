---
layout: post
title: "The Builder Pattern"
date: 2020-03-26 21:58:58 GMT
categories: swift design pattern
---

Sometimes building complex objects requires a lot of boilerplate code. Sometimes is just a matter of adjusting a large amount of parameters. 

There is a design pattern that let us abstract away the construction of such objects, by using an intermediate representation. 

Lets pretend we need to pack for a short trip. We can compile a list of things. We can encode such list in a struct. 

Struct in Swift have, among many other benefit, a constructor automatically generated for us. Lets leverage that, and the fact that they are value type to build our `PackingList` 

```
struct PackingList {
    let heavyCoat: Int
    let rainJacket: Int
    let pullover: Int
    let activewearCoat: Int
    let thermalUnderwear: Int
    let leggins: Int
    let darkJean: Int
    let sweater: Int
    let waterproofSnowBoots: Int
    let socks: Int
    let mittens: Int
    let beanie: Int
    let sunglass: Int
    let hat: Int
    let bathingSuit: Int
    let sunscreen: Int
    let handSanitizer: Int
}
```

Creating a new list would be labour intensive, we will need to gather all the params at a creation point. We could, also, let our type to be mutable, and define default values for every parameter. That would defeat the intrinsic safety that unmutability grant us. Builder is there to help us. 

We are going to add an extension to our `PackingList` to prevent polluting the interface. 

In this extension, we are going to define a helper class, called `Builder`. 

```
extension PackingList {
    class Builder {
    }
}
```

We use a class here to chain setters, as we are going to see shortly. 

Inside our helper class we define, our ivars, with a default value. 

```
private var heavyCoat: Int = 0
```

And the setters: 

```
func set(heavyCoat: Int) -> Builder {
  self.heavyCoat = heavyCoat
  return self
}
```

As we can see, the setter, returns `self`, so we can chain a bunch of them at once. That's why we use a reference type here. 

```
builder.set(heavyCoat: 0).set(rainJacket: 1)
```

Let's now consider the case of default packing, for summer. We can easily set default values in a helper function like:

```
func packForSummer() {
  self.set(heavyCoat: 0)
      .set(rainJacket: 1)
      .set(pullover: 0)
}
```

We can even go a setp further, define an `enum` to properly instruct default packing:

```
enum Weather {
  case summer
  case winter
}

func pack(for weather: Weather = .summer) {
  switch weather {
    case .summer:
      self.packForSummer()
    case .winter:
      self.packForWinter() 
  }
}
```

These two are just helpers, but now, let's define the most important one, the actual point where we build the `PackingList`: 

```
func build() -> PackingList {
  let packingList = PackingList(heavyCoat: self.heavyCoat,
                                rainJacket: self.rainJacket,
                                pullover: self.pullover,
                                activewearCoat: self.activewearCoat,
                                thermalUnderwear: self.thermalUnderwear,
                                leggins: self.leggins,
                                darkJean: self.darkJean,
                                sweater: self.sweater,
                                waterproofSnowBoots: self.waterproofSnowBoots,
                                socks: self.socks,
                                mittens: self.mittens,
                                beanie: self.beanie,
                                sunglass: self.sunglass,
                                hat: self.hat,
                                bathingSuit: self.bathingSuit,
                                sunscreen: self.sunscreen,
                                handSanitizer: self.handSanitizer)
  return packingList
}
```

Now, we can easily construct a `PackingList`, either for winter or summer:

```
let summerBuilder = PackingList.Builder()
summerBuilder.pack(for: .summer)
let summerPackingList = summerBuilder.build()
assert(summerPackingList.bathingSuit == 1)

let winterBuilder = PackingList.Builder()
winterBuilder.pack(for: .winter)
let winterPackingList = winterBuilder.build()
assert(winterPackingList.bathingSuit == 0)

```

Or we can easily customise the builder to properly build the list we want: 

```
let customSummerBuilder = PackingList.Builder()
customSummerBuilder.pack(for: .summer)
customSummerBuilder.set(bathingSuit: 2)
let customSummerPackingList = customSummerBuilder.build()
assert(customSummerPackingList.bathingSuit == 2)
```

A full implemetation can be found in [Gist](https://gist.github.com/volonbolon/c9b769ffc11ab93075668745a7bc0b71)