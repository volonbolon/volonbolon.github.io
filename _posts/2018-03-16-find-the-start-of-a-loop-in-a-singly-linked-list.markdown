---
layout: post
title: "Find the start of a loop in a singly linked list"
date: 2018-03-16 13:50:07 GMT
tags: cs challenges data structure algorithm
---

We [already know](https://iamvolonbolon.tumblr.com/post/168327385830/loops-in-a-singly-linked-list) how to detect loops in a singly linked list. Now, we are going to detect the node where the loop starts. 

It’s easy to see that if we have two pointers moving one at twice the speed than the other, and if they start at the same time at the beginning of the loop, then they will found each other again at the very same point (with the faster point completing two laps and the slower just one). 

What happens if, instead, the head of the singly linked list is not where the loop start. In that case, the slower pointer will give the faster one an advantage of k nodes, where k is the number of nodes from the head to the loop start. But where will the two pointers meet? They will meet k nodes *before* the end of the loop. Why? Well, by then, the slower pointer would have to walk `n - k` steps into the loop, and the fast one would have done `k + 2(n - k)`

![Loop](https://i.imgur.com/OQoiYEG.gif)

Once we get the two pointers at the same node, and because we know they are k nodes away from the finish/start of the loop, all we need to do is to reset one of the pointers to the head of the linked list and move the two at the same speed these time. They will meet again at the start of the loop. 

A complete implementation of the algorithm is in [Gist](https://gist.github.com/volonbolon/5bc6d6783eb0a2af9278b7a76088f5d0)