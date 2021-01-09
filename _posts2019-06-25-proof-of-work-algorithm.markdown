---
layout: post
title: "Proof-of-Work Algorithm"
date: 2019-06-25 22:44:54 GMT
categories: blockchain
---

To prevent rogue miners to spoof blocks and bend the chain in its favor, blockchain requires miners to satisfy a computational task hard enough that renders the whole doctoring increasingly expensive. Such a task is the *proof of work*. 

## Hashcash
Many cryptocurrencies, including Bitcoin, implements a proof of work system inspired by [Hashcash](https://en.wikipedia.org/wiki/Hashcash). 

At any given point, the network is configured with a certain level of difficulty. Each miner competing to generate a block will need to compute a hash that satisfies this difficulty. 

Now, we should remember that it is computationally unfeasible to produce two hashes with different inputs that produce the same output (a *collision*). An thus, it is also impossible to select and input in such a way to produce the desired hash, we should rely on brute force. And that's the core of the algorithm. Miners are required to come up with a value, known as *nonce*, that when combined with the rest of the block data, will produce a hash with the expected patron. In cryptography, a nonce is a value that can be used just once per communication. Since any unique nonce, when combined with the block data, will generate a unique hash, it is effectively used only once. 

Specifically, the task is to find a hash that starts with as many zeros as required by the difficulty level, ie. if the difficulty level is 8, then miners will need to come up with a hash that starts with eight 0. 

Finding such a hash becomes exponentially harder as difficulty increases. 

To come up with the nonce is hard, but, given the data and the nonce, is really cheap to compute the hash to validate it. 

### What's in a name
From the perspective of an observer, to come up with a hash with a given fingerprint can be assumed as the amount of work the miner devoted to the task. For instance, if the output of the hash function is evenly distributed (and it should be), we should expect to find a result that satisfies a difficulty level 1 (a hash that starts with a 0) once every 16 hashes. The miner would need to produce, on average, 16 different nonces to come out with the required hash. The input (the block data plus the nonce), constitutes the *proof* that a certain amount of *work* has been done.  

For instance the python code


```
import hashlib
for nonce in range(16):
  input = f"proof of work {nonce}"
  hash = hashlib.sha256(input.encode()).hexdigest()
  print(f"{input} => {hash}")
```


Would produce 


```
proof of work 0 => b0140183875afe8cb70aeb28768df900141496872fb80ddedc46867795a59c85
proof of work 1 => ca326d08692767df1ebaabd7110b2a20ef4a44b98357ab0064cb6512d4fdeb82
proof of work 2 => b8b980a3e0208bccdbeedcf3de82264e8b347e3682a2be00e2ee22accd95c8d6
proof of work 3 => 0f824adceeb31e9be937a58ef921bc716d9b9e31268547957f250cc11c5210ca
proof of work 4 => bfe6cda02cffd7571b0761e0a5fbc241ecf032d44223b70e855a6f4e62799299
proof of work 5 => f23fac3c20bac4142274ad9fbd147b6b9fcb9639f67890b1cb5e14c3a170b63f
proof of work 6 => 541dc25f1cfb360165399290bff83ec6a225b485db7fcebac5d8fec164364459
proof of work 7 => 93ea67585c69f22745eb2d462395e05ba6e8653654503967cf3f2e4de7ce9c9b
proof of work 8 => eedb1e8764c539d3ef0ef542bf5b966d8c68607aff7cf43f0140a35ff8e4e07c
proof of work 9 => 16e6a5f62cfc73d738c72e472de9b409a3209095243a45b4d030d848051963a2
proof of work 10 => 8352873cfcd2983c148cfe926bcd8cbfd3b16e861e6b0b95d3a042c504068234
proof of work 11 => 823de40d3c9cda98e8d5370c0e0a85ead14f71670dd61c9eedbcc2d33dd80939
proof of work 12 => d8657230b0aa16bd54caf922b71c0e9b6e7d48cb410bf0593a797ee68d3230c2
proof of work 13 => 32ff5b1a19605085000d8e4e12063b87bd6575558de9c9bec0157737891843b7
proof of work 14 => bd2bc7af1d6d45015cc808a10e5103f1bf823c7a1d9f96339833dbc9dc0f7ae3
proof of work 15 => ef82efc88dac6b50fd992bf9223ad67fdecd1aa279457cbb2bfaf5a7872ee533
```


In this particular case, the winning nonce would be `3`. 

### Difficulty

By adjusting the difficulty level the system can fix the rate of mining. Too many nodes competing? Let's increase the difficulty level. For example, Bitcoin network is configured to let miners create a new block every 10 minutes in average, and to keep the rate, constantly adjusts the difficulty level. 

## Fifty-one percent attack

If a rogue node has at least 51% of the computational power of the network, in theory, can replace the chain with a doctored version, long enough for other nodes to accept, including all the required proof of work puzzles. That's the theory, in reality, such a scenario would be so expensive that it would be absolutely ridiculous to try to manipulate the chain that way. 

Controlling 51% of the computation power of the network does not guarantee success to a malicious attack, it is just the point where the event is likely to happen. 

As Satoshi Nakamoto summarize in the bitcoin original [whitepaper](https://bitcoin.org/bitcoin.pdf)

> Proof-of-work is essentially one-CPU-one-vote. The majority decision is represented by the longest chain, which has the greatest proof-of-work effort invested in it. If a majority of CPU power is controlled by honest nodes, the honest chain will grow the fastest and outpace any competing chains.

On top of that, the attacker would be ruining the system he is trying to benefit from. After all, the confidence in the system would be forever lost, and, in the case of a cryptocurrency, its value would decline rapidly. Back to the bitcoin seminal whitepaper, Nakamoto explains: 

> If a greedy attacker is able to assemble more CPU power than all the honest nodes, he would have to choose between using it to defraud people by stealing back his payments or using it to generate new coins. He ought to find it more profitable to play by the rules, such rules that favor him with more new coins than everyone else combined, than to undermine the system and the validity of his own wealth.

For a brief period in 2014 Ghash.io achieved 51% of the computational power of Bitcoin. Obviously, the entire network was much smaller back then. To Ghash.io credit, they almost immediately relinquished 10 percent and asked the community to voluntary limit themselves to 40 percent.

[Code](https://github.com/volonbolon/turbo-octo-memory/commit/9ba395a0ae71436d1493948b6e387f85e82e31ff)