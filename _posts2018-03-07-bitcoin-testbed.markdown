---
layout: post
title: "Bitcoin testbed"
date: 2018-03-07 18:12:27 GMT
categories: bitcoin blockchain
---

Creating an experimental Bitcoin testbed is not trivial. Or is it? 

Tons of GB to store the blockchain (ok, yes, you can prune it, but first you have to download the entire thing), harder and harder algorithms to mine coins, incrementing cost for tests (after all, we are talking real money). Is a little bit intimidating. But the truth is that that applies just to `mainnet`. 

Besides `mainnet`, Bitcoin Core also has `testnet` and `regtest`. 

### testnet
This is the test network that runs in parallel with `mainnet`, except that the value of these coins is negligible. It exists to experiment with a blockchain that won’t harm the `mainnet`, e.g. with new features to Bitcoin Core. To prevent users to accidentally shift coins between `testnet` and `mainnet` the two runs on different TCP ports, and they have different genesis blocks. 

But the thing is that `testnet` is still a little daunting. Is difficult to mine new coins, and with a height larger than 1.25 million is not light. 

### regtest
`regtest` to the rescue. The “Regression Test” mode allows mining with practically zero difficulties and is initially disconnected from any existing network. It also has its own genesis block, and when you start your client, no peers are registered, and nothing is initially mined by anyone. You don’t need to carry a hard drive a 4TB hard drive to store the blockchain because it just contains the genesis block. The data directory is around 17MB (compare that with 150GB and growing from the `mainnet`). 

## Docker 

Using the excellent bit-core image provided by [ruimarinho](https://hub.docker.com/r/ruimarinho/bitcoin-core/) is really easy to setup a `regtest` instance.

```
> docker run --rm -it ruimarinho/bitcoin-core -printtoconsole -regtest=1 -rpcallowip=172.17.0.0/16 -rpcpassword=bar -rpcuser=foo
```

Now, if we want to explore the API trough cURL, we need to setup `rpcauth`. You can either do this yourself by constructing the line with the format `<user>:<salt>$<hash>` or use the official [rpcuser.py](https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py) script to generate this line for you. 

```
> curl -sSL https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py | python - volonbolon
String to be appended to bitcoin.conf:
rpcauth=volonbolon:1c1410ebdb3d3f7a368878c3c7b4a897$aae4200cf5b23db3dea907113ba1ee085a9733d0e41fe71a942549e2dec61b69
Your password:
Ep9FQjLUNbjw-MJPqKaGYWZchBPnfKHgAT1O7d-WiAQ=
```

Now that you have your credentials, you need to start the Bitcoin Core daemon with the `-rpcauth` option.

```
docker run --rm --name bitcoin-server -it ruimarinho/bitcoin-core -printconsole -regtest=1 -rpcallowip=172.17.0.0/16 -rpcauth='volonbolon:1c1410ebdb3d3f7a368878c3c7b4a897$aae4200cf5b23db3dea907113ba1ee085a9733d0e41fe71a942549e2dec61b69'
```

Now, we need to expose the ports. Ports can be exposed by mapping all of the available ones (using -P and based on what EXPOSE documents) or individually by adding -p. This mode allows assigning a dynamic port on the host (-p <port>) or assigning a fixed port -p <hostPort>:<containerPort>.

```
> docker run --rm -it -p 18443:18443 -p 18444:18444 ruimarinho/bitcoin-core -printtoconsole -regtest=1 -rpcallowip=172.17.0.0/16 -rpcpassword=bar -rpcuser=foo
```

And we are done: 

```
> curl -X "POST" "http://127.0.0.1:18443/" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -u 'foo:bar' \
     -d $'{
  "jsonrpc": "1.0",
  "id": "1",
  "method": "getnetworkinfo",
  "params": []
}'

HTTP/1.1 200 OK
Content-Type: application/json
Date: Wed, 07 Mar 2018 14:13:02 GMT
Content-Length: 599
Connection: close

{
  "result": {
    "version": 160000,
    "subversion": "/Satoshi:0.16.0/",
    "protocolversion": 70015,
    "localservices": "000000000000040d",
    "localrelay": true,
    "timeoffset": 0,
    "networkactive": true,
    "connections": 0,
    "networks": [
      {
        "name": "ipv4",
        "limited": false,
        "reachable": true,
        "proxy": "",
        "proxy_randomize_credentials": false
      },
      {
        "name": "ipv6",
        "limited": false,
        "reachable": true,
        "proxy": "",
        "proxy_randomize_credentials": false
      },
      {
        "name": "onion",
        "limited": true,
        "reachable": false,
        "proxy": "",
        "proxy_randomize_credentials": false
      }
    ],
    "relayfee": 0.00001000,
    "incrementalfee": 0.00001000,
    "localaddresses": [],
    "warnings": ""
  },
  "error": null,
  "id": "1"
}
```