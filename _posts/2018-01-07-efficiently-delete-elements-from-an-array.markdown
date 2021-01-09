---
layout: post
title: "Efficiently delete elements from an Array"
date: 2018-01-07 14:07:10 GMT
categories: cs challenges data structure algorithm swift
---

The classical setup for the problem is “delete characters from a string”. In the case of Swift, as we [already saw](https://iamvolonbolon.tumblr.com/post/169266108755/reverse-words-in-a-string), strings are not just characters arrays (like in C). So, let’s rephrase the problem to *Efficiently delete elements from an Array*. Where the `elements` to be deleted are also provided as an Array. 

The brute force solution would be to create a buffer and to traverse both, the array where we need to find the elements to delete, and the one containing the elements to be deleted, and only copy to the buffer those elements not marked to be deleted. 

The running time would be O(n*m), where n is the numbers of elements in the original array, and m the number of elements in the array with elements to be deleted. Here we can improve the running time by transforming the arrays with elements to be deleted to a hash table. Lookup for a hash table is, on average, constant. If we need to traverse m just once to build the hash table, complexity is reduced to O(n+m). There nothing we can do with n. After all, we do need to traverse the original array to check each of its values. 

Now, let’s think about the whole buffer array. Let’s consider the array

```
[1,2,3,4]
```

From where we want to delete, let’s say, `[1,3]`. 

If instead of copying the valid elements to a new array, we swap elements within the array, pushing the unwanted element to the beginning or the end, and then just slice the array, we can ditch the buffer altogether. In C, is easier to slice an array chopping the end. That’s not the case in Swift, but, just for the sake of uniformity, let’s agreed that we are going to shift the elements we want to keep to the beginning of the array (and then chop the unwanted elements from the end). 

In our example, we would need to shift `1` all the way to the end, or `4` all the way to the beginning. In the worst case, where we need to delete all the elements of the array, we would need to shift the last element n positions to the beginning, the element before that n-1 positions and so on. We would we running in O(n^2). 

Perhaps the buffer idea is not so bad after all. With a buffer, we just need to copy the elements we want to keep to the buffer, and that’s all. We can forget about them. Each time we find an element we want to keep, we move it to the buffer, and we keep moving along the original array. Again, once copied, we can forget about the characters. And the cardinality of the buffer will tell us how many elements we preserved up to any given point. 

It turns out, that’s all we need, to know at any given point if we want to keep the element we are inspecting. We don’t really need the buffer, we just need its cardinality. And we can switch elements within the same array, using the *buffer cardinality* as an index. We can actually use the array we are inspecting as the buffer itself. Once we are done, we slice it up to the buffer cardinality. 

Let’s revisit our example. We start by looping `[1,2,3,4]`. We find that we don’t want `1`, we move to the next element. `2` is one that we want to keep. If we would using a buffer, it would be empty at this point. Its cardinality would be zero. So, let’s switch the element at position 1 (the element currently inspected), with the one at position 0 (the *last* element of the buffer, if you want). The array when leaving the current loop would be `[2,1,3,4]`. We are currently in position 2, which points to element 3, we don’t want it. Move to position 3. The 4 is an element we want to keep. So we move it to the *last* element of the buffer, that is, 1. The array when leaving the loop is `[2,4,3,4]`, and the buffer cardinality is now 2. We reached the end of the array to inspect. All we need to do is to *cut* the buffer, i.e. the first two elements of the array: `[2,4]`

Here the implementation where an Array extension returns a slice of itself. 

```
extension Array where Element:Hashable {
    mutating func deleteElements(_ elements:[Element]) -> ArraySlice<Element> {
        var toDelete: [Element: Bool] = [:]
        for e in elements {
            toDelete[e] = true
        }
        var dst = 0
        var src = 0
        for i in self {
            let found = toDelete[i, default: false]
            if !found {
                self[dst] = self[src]
                dst += 1
            }
            src += 1
        }
        return self[..<dst]
    }
}
```

Remember that, an array slice is just a view to the array. We are not copying or creating anything expensive here. 

As usual, the complete implementation along with the same sample is in [GistHub](https://gist.github.com/volonbolon/f33014f7211c7c8bb44cc8daa243b7e4). 