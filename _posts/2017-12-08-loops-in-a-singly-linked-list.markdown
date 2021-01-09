---
layout: post
title: "Loops in a singly linked list"
date: 2017-12-08 16:57:49 GMT
categories: cs challenges
---

Given a Singly Linked List, the challenge is to find out if the tail is connected to the head, i.e. if we are dealing with a loop. 

The (almost) brute force solution would be to traverse the list, starting at, and save the ids of each node into a data structure like a Set. For each new node, we check if that is already stored in the set, in which case, we are dealing with a loop. 

Set should be able to return if it contains a given value in *O(1)*, *O(n)* being the worst case. So, adding that to the *O(n)* time complexity of traversing the list in the first time, on average we should expect to get done in *O(n)*. Not bad. 

But, what if space complexity is a problem? Then we can traverse the list with two pointers, one traveling faster than the other. If, at some point, the faster overlaps the slower, then we know we have a loop. And we don’t need to maintain a Set or a hash table to keep track of the visited nodes. And we are still at a time complexity of *O(n)* (with no risk to fall down into *O(n^2)*)

```
extension LinkedList {
    func detectCycle() -> Bool {
        var slow:NodeType? = self.head
        var fast:NodeType? = self.head
        var found = false
        while slow != nil && fast != nil {
            slow = slow?.next
            fast = fast?.next?.next
            if let s = slow, let f = fast {
                if s == f {
                    found = true
                    break
                }
            }
        }
        return found
    }
}
```

The entire example can fe found in the [linked](https://gist.github.com/volonbolon/9a950782460bd7256385c98a87848847) playground. 