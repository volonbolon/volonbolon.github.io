---
layout: post
title: "Reverse a Singly Linked List "
date: 2017-12-08 19:58:40 GMT
categories: cs challenges
---

<p>And, since we already have a <a href="https://iamvolonbolon.tumblr.com/post/168327385830/loops-in-a-singly-linked-list">singly linked list</a>, let’s try to reverse it.</p>
<p>Let’s consider the very basic structure of a linked list to help us find a solution. In essence, we have just a chain of nodes, each one pointing pointing to the next, and the special case is the last one in the chain (the tail). We indicate the special case by setting the <em>next pointer</em> to <code>nil</code></p>
<pre><code>Node(1)-&gt;Node(2)-&gt;Node(3)-&gt;Node(4)-&gt;Node(5)-&gt;Node(6)-&gt;Node(7)-&gt;Node(8)-&gt;Node(9)-&gt;Node(10)-&gt;nil
</code></pre>
<p>The brute force solution would be to traverse the entire linked list until we get to the tail, preserving each step in a data structure (perhaps a stack) and then, produce a new list with the items in the reverse order. Obviously, we would be increasing the space complexity. But let’s get back to the print-out. To reverse the linked list all we need to do is to flip the direction of the arrows.</p>
<pre><code>    func reversed() {
        guard let head = self.head else {
            return
        }
        var previousNode:NodeType? = nil
        var currentNode:NodeType? = head
        var nextNode:NodeType? = nil

        while currentNode != nil {
            // if there is a `next` node, we keep moving
            if let next = currentNode?.next {
                nextNode = next
            } else {
                // but if we are at the last node, we reset the head of the list
                self.head = currentNode
                nextNode = nil
            }
            // Let's change  the direction of the next pointer arrow
            // In the first loop, `previousNode` marking the sentinel of the reversed list
            currentNode?.next = previousNode
            // and now, prev is pointing to current, and we move on
            previousNode = currentNode
            currentNode = nextNode
        }
    }
</code></pre>
<p>A working <a href="https://gist.github.com/volonbolon/f220ddb3a9600a6643f73420a846086a">playground</a> is waiting in gist.</p>