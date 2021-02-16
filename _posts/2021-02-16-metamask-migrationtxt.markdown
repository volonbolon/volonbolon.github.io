---
layout: post
title: "Metamask migration"
date: 2021-02-16 16:57:20 GMT
tags: ethereum metamask web3
description:
---
In January 2021 Metamask introduced a number of breaking changes. 

## Stopped web3 injection
Metamask was injecting a pretty old version of Web3 (0.20.x), which is no longer maintained. Since web3 version 2.0.0 is already scheduled for release, to migrate to version 1.x.x would have been a waste of time, which would have introduced a number of breaking changes. And more breaking changes would be required when introducing 2.x.x. Metamask decided not to dance that particular waltz, injecting only a dummy web3 designed to warn clients. 

Instead, we are expected to use `window.ethereum` directly, which is fine, since web3 is already mapping most of the requests to RPC, and we can call them with `window.ethereum.request()`

```
const accounts = await ethereum.request({ method: 'eth_accounts' });
```

## API Changes
`window.ethereum` also introduced a few API changes. 

* Padded chain ID
* Replace `chainIdChanged` with `chainChanged`
* Remove experimental methods: 
	* `ethereum._metamask.isEnabled`
	* `ethereum._metamask.isApproved`
* Remove `ethereum.publicConfigStore`
* Remove `ethereum.autoRefreshOnNetworkChange`

The chain ID returned by `eth_chainId` is now returning the value without padding it. Instead of 0x01, it is not returning 0x1. 

The two helpers introduced to let web3 clients query if the user had granted access to his account are now removed. We should call `wallet_getPermissions` and check for `eth_accounts` permissions.

`ethereum.publicConfigStore` is a special case. Despite its name, it was never meant to be a public API. The big problem is that there are some third-party libraries using it. The good news is that both, ethers, and web3 are not using the API (at least the last version of both of them).  