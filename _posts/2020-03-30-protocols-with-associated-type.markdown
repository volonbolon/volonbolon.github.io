---
layout: post
title: "Protocols with Associated Type"
date: 2020-03-30 20:39:01 GMT
tags: Swift protocol pop
---

Swift is type-safe, all types must be defined at compilation time.

We can easily define a protocol that requires a given type. 

```
protocol MyProtocol {
    var myProperty: String { get set }
}
```

And then, we can then adopt the protocol in other types. 

```
struct MyStruct: MyProtocol {
    var myProperty = "this is my property"
}
```

So far so good. We have a type that adopts a protocol. The compiler knows exactly what to do. Everyone is happy. 

We can leverage the power of generics announcing the compiler that we are going to define the actual type later, at use time. For that, we use `associatedtype`.

```
protocol MyGenericProtocol {
    associatedtype myType
    var myProperty: myType { get set }
}
```

Here, `myType` is a placeholder that will be defined by the type implementing the protocol. 

Let's use it to feed some animals. 

```
protocol Food {}
protocol Animal {
    associatedtype FoodType: Food
    func eat(food: FoodType)
}
```

Here we are telling the compiler that animals eat food. We don't know yet the actual type of food each animal eats, we are going to define that later. Or now. 

```
struct Grass: Food {}

struct Cow: Animal {
    func eat(food: Grass) {
        print(food)
    }
}
```

`Grass` is a concrete implementation of `Food`, `Cow` is a concrete implementation of `Animal`, and, as we can see, it happens to eat `Grass`. 

### First-class citizen
In swift, protocols are first-class citizens. We can pass them around as return type, or as function attributes. But, if we try something like 

```
func feed(animal: Animal) { }
```

We get the following error: 

```
error: protocol 'Animal' can only be used as a generic constraint because it has Self or associated type requirements
```

The thing is that we can pass protocols as types, as long as they don't contain an associated type. 

But we can use strait generics to let the compiler knows that we want to feed animals. 

```
func feed<A: Animal>(animal: A) {
    if let cow = animal as? Cow {
        let grass = Grass()
        cow.eat(food: grass)
    }
}
```

Here we just define a generic, requiring it to be a concrete instantiation of `Animal`, and that's that.