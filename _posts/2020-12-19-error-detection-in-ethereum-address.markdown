---
layout: post
title: "Error detection in Ethereum Address"
date: 2020-12-19 13:26:17 GMT
categories: ethereum blockchain crypto
---

Sometimes, you find beauty in the simplest thing. Let's consider how Vitalik Buterin and Alex Van de Sande introduced a checksum in Ethereum addresses. 

First, let's remember that, unlike Bitcoin, Ethereum addresses do not incorporate a checksum. It was considered unnecessary. With [ICAP](https://github.com/ethereum/wiki/wiki/ICAP:-Inter-exchange-Client-Address-Protocol) support, and [Ethereum Name Service](https://ens.domains), humans were not supposed to interact with raw hexadecimal strings. These two techniques (ICAP, a convention compatible with [IBAN](https://en.wikipedia.org/wiki/International_Bank_Account_Number) and ENS), are far better than the Base58 + checksum approach used in Blockchain, nevertheless, the transitional checksum method introduced by Buterin is brilliant. 

In a typical hexadecimal representation of an address, we will have an average of 15 letters. 

Let's consider this address: 

```
0x8da82fcc73cbf2fb1a8c627ec2f0e3fcc3f06748
```

It contains 20 letters. Old wallets are case insensitive. `8da` is not different from `8DA`. That's important, because, it will allow us to modify the address to introduce a checksum without breaking these wallets. 

First, we will take the Keccak256 hash from the lowercased address without the `0x` prefix. 

```
>>> keccak_256(b'8da82fcc73cbf2fb1a8c627ec2f0e3fcc3f06748').digest().hex()[:40] # We only need the first 20 bytes of the hash
'bfc2e364e7ddd96192c76f7fd3d6816e4cafbf96'
```

Now, we put the original address and the Keccak hash together, and whenever we found a letter in the address, we check the corresponding character in the hash. If it happens to greater than 8, we capitalize the letter, otherwise, we keep the lowercase. 

```
8da82fcc73cbf2fb1a8c627ec2f0e3fcc3f06748
bfc2e364e7ddd96192c76f7fd3d6816e4cafbf96
```

For instance, the second and the third character in our original address are letters, and the corresponding characters in the hash are both greater than 8, which means that we should uppercase these two characters. `8DA…`. The final checksum encoded address would be: 

```
8DA82fcc73CBF2fb1a8c627EC2F0E3fCc3F06748
```

## Detecting Errors 
Let us now introduce a tiny mistake to simulate a misread. For instance, the 6th character right to left in the checksum encoded version of our address is `F`, easy to confuse with, let's say, `E`. 

Fortunately, our wallet is EIP-55 complaint and seeing that the address contains both, lower and uppercase chars, it will create the checksum to compare. Because the hash function is highly sensitive to changes in the input, the hash itself is completely different (we changed only one bit, E and F are one bit apart)

```
25d14f3dff97d80217264e3de353a2cb0fde9add
```

And the checksum produced is all wrong: 

```
8dA82FcC73CbF2fb1a8c627EC2f0E3FCc3E06748
```

You can take a look at a simple python implementation [here](https://gist.github.com/volonbolon/11d13af2a5d168bc12cd4bc0ad0066d0)