---
layout: post
title: "Opaque return types in Swift"
date: 2021-01-10 23:40:29 GMT
tags: swift iOS foundation
description: "Swift 5.1 introduces a new type of return type that allows the compiler to perform a stricter control, and gives the programmer more freedom"
---
Abstraction is good. We in swift have tons of features to let us abstract work. But up until Swift 5.1, we would be required to define the return type of a method. 

```
func foo() -> Int {
    return 42
}
```

True, we can leverage `Generics` to give us some slack. 

```
func foo<T: Numeric>(arg: T) -> T {
    return arg + 42
}

foo(arg: 42) // 84
foo(arg: 32.5) // 74.5
```

But, that's not a complete abstraction. After all, the caller is defining the return type. In the first case, we will get back an int, and in the second, a double. 

With a protocol, we can introduce more abstraction. 

```
protocol Bacteria {
    func split() -> [Bacteria]
}

struct GramPos: Bacteria {
    func split() -> [Bacteria] {
        return [self, GramPos()]
    }
}

extension GramPos: CustomStringConvertible {
    var description: String {
        "GramPos"
    }
}

struct Cyanobacteria: Bacteria {
    func split() -> [Bacteria] {
        return [self, Cyanobacteria()]
    }
}

extension Cyanobacteria: CustomStringConvertible {
    var description: String {
        "Cyanobacteria"
    }
}

func petriDish() -> Bacteria {
    return Cyanobacteria()
}

let b = petriDish()
print(b)
```

Anyone calling to `petriDish()` knows there is a bacteria coming back, don't know the actual type. And even worse, if we want to check the specific type, conforming to `Equatable`

```
protocol Bacteria: Equatable {
    func split() -> [Self]
}
```


We get the dreaded *Protocol 'Bacteria' can only be used as a generic constraint because it has Self or associated type requirements*. The reason? The **Equatable** protocol has to compare two instances of itself (`Self`) to see whether they are the same, but Swift has no guarantee that the two equatable things are remotely the same. 


```
error: MyPlayground.playground:44:21: error: protocol 'Bacteria' can only be used as a generic constraint because it has Self or associated type requirements
func petriDish() -> Bacteria {
                    ^

error: MyPlayground.playground:58:4: error: binary operator '==' cannot be applied to two 'Bacteria' operands
b1 == b2
~~ ^  ~~
```

The compiler doesn't really know what type of bacteria we are dealing with. It knows we are dealing with objects that conform to `Bacteria`, nothing more. 

By returning an **opaque type**, we as the callers could enjoy the flexibility of a Protocol, but the compiler knows the type of the returned object, and thus, can perform comparisons (and do whatever it needs to do in the confidence of knowing what it is dealing with)

```
let b1 = petriDish()
let b2 = petriDish()

b1 == b2 // true
```
 
An important thing to remember is that functions with opaque return types must always return *one specific type*. If for example, we tried to randomly create Cyanobacteria or GramPos then Swift would refuse to build our code because the compiler can no longer tell what will be sent back.

```
func petriDish() -> some Bacteria {
    if Bool.random() {
        return Cyanobacteria()
    }
    return GramPos()
}

/* 
error: MyPlayground.playground:44:6: error: function declares an opaque return type, but the return statements in its body do not have matching underlying types
func petriDish() -> some Bacteria {
     ^

MyPlayground.playground:46:16: note: return statement has underlying type 'Cyanobacteria'
        return Cyanobacteria()
               ^

MyPlayground.playground:48:12: note: return statement has underlying type 'GramPos'
    return GramPos()
           ^
 */
```