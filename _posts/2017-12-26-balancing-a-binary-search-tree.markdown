---
layout: post
title: "Balancing a Binary Search Tree"
date: 2017-12-26 18:53:15 GMT
categories: cs challenges data structure algorithm
---

<p>A <a href="https://en.wikipedia.org/wiki/Binary_search_tree">Binary Search Tree</a> is a container data structure, that keeps its keys always sorted. Any given node is greater than or equal to any key stored in the left sub-tree, and less than or equal to any key stored in the right sub-tree.</p>
<p>Search, Delete and Insert operations are proportional to the height of the tree. In average, binary search complexity is O(log(n)), but in the worst case, we are basically dealing with a singly linked list, and search is O(n).</p>
<p><a href="https://image.ibb.co/dVK40R/input.png"><figure class="tmblr-full" data-orig-height="202" data-orig-width="371" data-orig-src="https://image.ibb.co/dVK40R/input.png"><img src="https://66.media.tumblr.com/57ebe67d17354252639e4cd8ec9509bf/79802b1681e87d1c-66/s540x810/5c51125b848577a28f03a34a0f2ef0e35d720213.png" alt="Unbalanced BST" data-orig-height="202" data-orig-width="371" data-orig-src="https://image.ibb.co/dVK40R/input.png"></figure></a></p>
<p>So, as you can see, in order to obtain best results, BSTs should be balanced. And, by balanced, we mean that the height should be log(n).</p>
<p>So, let&rsquo;s start with the degenerate case:</p>
<p><a href="https://thumb.ibb.co/gzp1Gb/degenerate.png"><figure data-orig-height="180" data-orig-width="180" data-orig-src="https://thumb.ibb.co/gzp1Gb/degenerate.png"><img src="https://66.media.tumblr.com/da638e094b8d4a262b10f6dc8429ea54/79802b1681e87d1c-1f/s540x810/293b2568263c0a5df4490a96fdaaf34629826d27.png" alt="Degenerate BST" data-orig-height="180" data-orig-width="180" data-orig-src="https://thumb.ibb.co/gzp1Gb/degenerate.png"></figure></a></p>
<p>Is easy to see that, search here will take <code>O(n)</code>, and is also easy to see that if we can <em>rotate</em> the whole tree around the middle (4 in this case), then we would be reducing the height in half. It turns out that that&rsquo;s all we need. We need to convert the unbalanced BST to the degenerate case, and then perform rotations around the middle point of each subtree.</p>
<p>But how to get the degenerate case? Again, fairly easy. By the very structure of the BST, if we traverse the tree in-order, the resulting data structure is the degenerate tree (or a representation of it. For instance, in the <a href="https://gist.github.com/volonbolon/360149920f8adecee98e025602e053f1">playground</a> attached, I&rsquo;ve chosen to dump everything to an array because after obtaining the list, we need to manipulate it accessing elements at random. A linked list or a tree will be enough, but an array is far better to access the element in the nth position. An important consideration, if we are dealing with a huge tree, then it would be better to manipulate the tree in place to save memory).</p>
<p>Once we have the degenerate tree, all we need to do is to keep rotating around the middle point of each sub-tree, until we are done. And the very best is that by the structure of any tree, recursion is almost mandatory.</p>
<pre><code>func sortedArrayToBST(_ sortedArray: [T], start: Int, end: Int) -&gt; BinarySearchTree&lt;T&gt; {
        if start &gt; end {
            return BinarySearchTree.empty
        }

        let middle = (start + end) / 2

        let left = sortedArrayToBST(sortedArray, start: start, end: middle - 1)
        let right = sortedArrayToBST(sortedArray, start: middle + 1, end: end)

        let root = BinarySearchTree.node(left, sortedArray[middle], right)

        return root
    }
</code></pre>
<p><a href="https://image.ibb.co/kPXA9w/output.png"><figure data-orig-height="117" data-orig-width="258" data-orig-src="https://image.ibb.co/kPXA9w/output.png"><img src="https://66.media.tumblr.com/d1128c36c94fcf060335c42ad1d224e6/79802b1681e87d1c-5f/s540x810/a6a468abf8d1994c098b7aa52dbffd19297e2181.png" alt="Balanced BST" data-orig-height="117" data-orig-width="258" data-orig-src="https://image.ibb.co/kPXA9w/output.png"></figure></a></p>