---
layout: post
title: "Blockchain Public Key Representations"
date: 2018-04-11 23:51:47 GMT
categories: blockchain crypto bitcoin
---

As we [already saw](https://iamvolonbolon.tumblr.com/post/172725515985/blockchain-keys), the public key is nothing but a point in the curve. The one we get after multiplying the generator the number of times defined by the secret. 

Now, it seems a waste of space to save the x and y coordinates when we know it is the solution to the equation `y^2 % p == (x^3 + 7)` If we know `x`, we can calculate `y`. Public keys are present in most transactions, and since each block can contain several hundred transactions, by cutting the required space for each public key, we are saving a lot of storage (and network traffic). That’s the reason for the compressed public keys. They are basically, just the x coordinate of the point. 

By *basically*, well, because since in the equation we have y squared, and the curve is symmetrical with respect to the x-axis, we need to somehow be able to know if y is positive or negative. We will use a prefix to distinguish positive and negative y. `02` indicates a positive y value. `03` a negative one. 

But remember, we defined the curved not over the Euclidean plane, but on the finite field of prime order *p*. Here, the y coordinate is always positive. We distinguish between odd and even. Even values correspond to positive values when working over the Euclidean plane. Which means, `02` is used for even values, while `03` is reserved for odd values. 

The old, uncompressed public keys are identified by the prefix `04`.

To sum this up. The public key, by definition, is an ordered pair representing a point in the curve. Old, uncompressed public keys are identified with the prefix `04` and contain both coordinates. The newer form to represent the public key is to use just the x coordinate, and the prefix is used not just to identify the key as a “compressed key“, but also to inform whether y is positive or negative.