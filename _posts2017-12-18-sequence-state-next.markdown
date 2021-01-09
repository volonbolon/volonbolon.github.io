---
layout: post
title: "sequence state next"
date: 2017-12-18 20:42:28 GMT
categories: swift tips undies
---

A really neat generic function is `sequence(state: next:)`. The best part is that it produces a sequence lazily by running a closure. ie, each step is computed. 

The function parameters are the initial `state`, to be passed to the closure, and the closure itself, which receives and `inout` `state` on each run. We can update the state on each cycle, adjusting it’s value if needed. The closure will be executer until `nil` is returned.

This is specially neat to, for instance, traverse the hierarchy of views. </p>

```
let seq = sequence(state: greenView) { (next: inout UIView) -> UIView? in
    let parent = next.superview
    next = parent != nil ? parent! : next
    return parent // The top most `parent` will be nil, stopping the execution of the closure
}
```

Lazy is good, and you can take a look at a quick example of `sequence(state: next:)` in [this gist](https://gist.github.com/volonbolon/0a04eab78fbc06d73ad6fac9e9748fc8) 