---
layout: post
title: "Escape vs Non-Escape closures"
date: 2017-12-20 23:49:59 GMT
tags: memory management tips implementation code swift
---

Let’s first try to understand what a escaping closure is. If a closure is passed as an argument of a function and is invoked after the function returns, the closure is *escaping*. Alternatively, if the closure is execute within the recipient scope, it’s *non-scaping*


```
var completionHandler:()->Void = {
    print("This is escaping, because it's invoked by a function after returns")
}

func funcExpectingEscapingClosure(completion:@escaping ()->Void) {
    closureRunner(completion: completion)
}

func closureRunner(completion: @escaping ()->Void) {
    completion()
}

funcExpectingEscapingClosure(completion: completionHandler) // This is escaping, because it's invoked by a function after returns
```

By default, closures passed as parameters of a function are marked as `@noescape`. If, for instance, we remove the classification from the signature of `funcExpectingEscapingClosure`, the system will default to `@noescape`, and the compiler is going to produce an error. 

```
error: MyPlayground.playground:4:31: error: passing non-escaping parameter 'completion' to function expecting an @escaping closure
    closureRunner(completion: completion)
```


Up to Swift 3 the default was to treat all closures as `@escape` , but thanks to the evolution proposal [SE-0103](https://github.com/apple/swift-evolution/blob/master/proposals/0103-make-noescape-default.md) Doing this, the compiler can optimize the code. For instance, if we know we are dealing with a `@noescape` closure, then is safe to release all the captured objects by the time the host function returns. But if we are dealing with a `@escaping` closure, then is really easy to create a reference cycle by just using `self`. 

```
class Child {
    func bar(completion: @escaping () -> Void) {
        DispatchQueue.main.async {
            completion()
        }
    }
}

class Parent {
    var child = Child()
    var x = 0

    func foo() {
        self.child.bar {
            /**
             By using `self`, we are capturing the parent instance,
             and passing it to the Child through the closure. 
             But the parent already contains a reference to child, neither child
             will be able to release parent, nor parent will release child. 
             */
            self.x = 42
            print(self.x) // 42
        }
        print(self.x) // 0
    }
}
```

[complete playground](https://gist.github.com/volonbolon/7a575b4d5c5a45d22601ca1ec9df8a68#file-reference_cycle-swift)

The easiest walk around is to capture the `self` as either `unowned` or `weak`. `unowned` is usually preferred because it marks the captured variable to be release along the capturer. `weak` is a little more dangerous, because we don’t really known when the captured variable could become `nil`. [code](https://gist.github.com/volonbolon/7a575b4d5c5a45d22601ca1ec9df8a68#file-reference_cycle_fixed-swift)