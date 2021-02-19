---
layout: post
title: "Truffle Customization.txt"
date: 2021-02-19 18:50:18 GMT
tags: ethereum truffle blockchain
description:
---

Truffle is a wonderful tool, it helps enormously to create and deploy contracts, and is configurable. 

The default destination for builds is `build/contracts/` which is fine unless you want to load them in Web3 to expose them in a DApp through React. Rect is going to complain that the file is outside the source folder. No biggie, we can move the build destination with: 

```
contracts_build_directory: path.join(__dirname, "src/abi"),
```

With the contract compiled, we can deploy easily to different testbeds and networks. First, we need to create the networks in *truffle-config.js*

```
module.exports = {
	networks: {
		development: {
     host: "127.0.0.1",
     port: 8545,
     network_id: "*", // Any network (default: none)
    },
    rinkeby: {
	    provider: function() {
        return new HDWalletProvider(<mnemonic>, "https://ropsten.infura.io/v3/<INFURA_PROJECT_ID>")
      },
      network_id: 3
    } 
	}
}
```

Then, we can migrate our contracts informing the desired destination.

```
truffle migrate --network rinkeby
```