---
layout: post
title: "Implicitly Required Initializers"
date: 2017-09-08 15:55:48 GMT
tags: iOS Swift protocol
---

An interesting consequence of `init`s defined in protocols (actually, any method, but let’s stick with `init`s) is that, by default, they are required. For a value type, that’s not a big deal. But for a reference type, it means that the class adopting the protocol not only has to define the `init`, but it also has to ensure the compiler that all its possible subclasses also will be also sporting the same initializer. Either by heritance, or by overriding, or by declaring the class as final (and thus, preventing inheritance)

```
protocol Polygon {
   init(sides:Int)
}

class Simple:Polygon {
   let sides:Int
   required init(sides:Int) {
       self.sides = sides
   }
}

class Convex:Simple {

}

final class Concave:Polygon {
   let sides:Int
   init(sides: Int) {
       self.sides = sides
   }
}

let triangle = Convex(sides: 3)
let pentagon = Simple(sides: 5)
let dart = Concave(sides: 4)
```