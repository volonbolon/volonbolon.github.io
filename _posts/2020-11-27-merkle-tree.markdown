---
layout: post
title: "Merkle Tree"
date: 2020-11-27 00:09:40 GMT
categories: crypto basic
---

They are a hash based data structure that generalize the concept of a hash list. Each leaf node is a hash of a block of data, and each non-leaf node is a hash of the concatenation of its children. Usually, they are build as binary trees, although thats not a requirement.

![Merkle Tree](https://9gnrkan899.s3-sa-east-1.amazonaws.com/merkle.png)

With them we can efficiently validate data across a network. 

At the most basic level, we can compare two trees for identity by comparing the root. The root is a representation of the whole data set. The change of any single leaf at the bottom bubbles up to the root. This is not only wonderful because it reduces the number of comparisons we need to perform, it is even better because it reduces the amount of data we need to dispatch over a network. Typically 32 bytes, instead of, perhaps, GBs worth of content. 

![Root Equality](https://9gnrkan899.s3-sa-east-1.amazonaws.com/merkle_equality.png)

But wait, if all we want is to check the integrity of a potentially huge file, why bother with a tree data structure. We can hash the file, are we are done. 

That's correct. If we only want to check the integrity of a data set, a hash is enough. We do build the tree because it is convenient to perform partial verifications. In a peer to peer network, we can verify subsets as we receive them. Even further, we might be interested only in a a subset, and we the tree structure, we can validate the portion we need. 

If we receive the root from a trusted source, then we can ask for the pieces from any source. We can later verify the integrity of the whole or a subset later. 

## Proof of membership
Let's pretend we want to check if `D` is part of a tree.

First, we need to get the path to the leaf we want to evaluate. Once we have that path, we need to produce a proof. Essentially, a road map to traverse the tree, rebuilding the portions that we need. 

![Proof of Membership](https://9gnrkan899.s3-sa-east-1.amazonaws.com/merkle_membership_2.png)

### Proof
To build up the proof, we can move down from the root, following the path to the leaf, and annotating our journey. We will find three types of nodes.

* A leaf
* A node in our path
* A sibling of a node in our path

A simple algorithm would demand a bitfield to annotate information about the node we are visiting, and a list to save node data that we will need later to compute the tree.

* If the node we are visiting is  a leaf, we enter a 1 into the bitfield and we add the node to the   list
* If the node is in our path, we push a 1 to the bitfield, and we calculate its value traversing left and right. 
* Otherwise we introduce a 0 to our bitfield, and we move back to the parent. 

Starting at the root `ABCDEEEE`, which obviously is in our path, we have: 

```
Bitfield = 0b1
Hashes = []
```

`ABCD` is in our path: 

```
Bitfield = 0b11
Hashes = []
```

`AB` is not in our path, let's add a zero to the bitfield and save `AB` to the hashes list

```
Bitfield = 0b110
Hashes = [`AB`]
```

Back to `ABCD`, we traverse to the right. `CD` is in the path.

```
Bitfield = 0b1101
Hashes = [`AB`]
```

Now we are at the leaves level, let's add both to the hashes

```
Bitfield = 0b110111
Hashes = [`AB`, `C`, `D`]
```

Since we completed the left branch of the root, we should go right, were we find a node which is not part of our path. 

```
Bitfield = 0b1101110
Hashes = [`AB`, `C`, `D`, `EEEE`]
```

### Membership validation
We have our proof, and we know that, since we have 5 leaves, the height of our tree is four[^1]. 

Now we can build a partial tree, and hopefully, find out that the root is the same (and thus, we will verify the integrity of the actual tree, and that the leaf under evaluation is, actually, part of it). 

![Membership Validation](https://9gnrkan899.s3-sa-east-1.amazonaws.com/merkle_membership_3.png)

If the bit extracted from the bitfield is 1, and we are not at the leaf level, we append an empty node (which value would be computed later) to the tree. If we found a 1, but we are at the leaves level, we pop the value from the hashes array. If the bit is zero, we also pop the value from the hashes list, and move back to the parent. All the way back until we find a node with a null right node, and then, we traverse that branch. 
In our case, we will end up with something like this. 

And by filling the empty nodes, we end up calculating the root to validate the entire tree. 
 
## Use cases 
Git use Merkle Trees to check for consistency across multiple repos.

Blockchain also use Merkle Trees to preserve block integrity. Each transaction is hashed, and a tree is computed. Altering a single transaction is unfeasible. 

Merkle trees can be used to check for inconsistencies in more than just files and basic data structures like the blockchain. Apache Cassandra and other NoSQL systems use Merkle trees to detect inconsistencies between replicas of entire databases.

[^1]: We can accommodate five leaves in a tree that will fit 8 leaves when perfectly balanced, and with that many leaves, we will have 15 nodes, and four levels