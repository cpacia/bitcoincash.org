---
layout: specification
title: Mining Protocol(s)
category: spec
date: 2019-04-24
activation: N/A
version: 0.1
---

Abstract
===============
This document introduces and defines two new protocols, a work protocol and a pool protocol, designed to make Bitcoin Cash mining
more efficient and more decentralized. It is based partially on the work of Matt Corallo[1] and on prior work of BCH developers.

Motivation
===============
Bitcoin Cash mining currently uses poorly designed and poorly documented protocols that date back to the early days of Bitcoin. A typical 
setup consists of a mining pool connecting to its instance of `bitcoind` to download a block template using the `getblocktemplate` RPC
and translating that template into work to send to the individual miners via the `stratum` protocol.

<p align="center"><img src="https://i.imgur.com/ToOmdZm.png"/></p>

`getblocktemplate` is a long polling RPC which is both inefficient and unreliable. It also sends a lot of unnecessary data and uses way
more bandwidth than is reasonable. 

`stratum` is a protocol for centralized mining by design. In this setup miners, despite owning the hardware, act only as dumb
hashers, without any visibility into and without any say in what they are actually mining. 

The resulting network topology puts an enormous amount of power into the hands of just a handful of highly centralized mining pools. With 
this power they can execute 51% attacks and steal coins from users, activate softforks against the wishes of users and the miners whose hardware 
they are controlling, and block protocol changes desired by the rest of the community. 

A saving grace to this point has been the knowledge that should pool operaters behave maliciously the individual miners would likely leave for
other pools. However, miners do not necessarily monitor the network 24 hours a day, and plenty of damage could still be done before miners had
the opportunity to switch pools, if they care to at all.

A better network topology would be one in which miners run their own full nodes and select their own transactions rather than outsourcing this
responsibility to a pool. In such a network the pool would still be responsible for aggregating and distributing the reward, but would not
play any role in transaction selection or the activation of protocol changes.

New Network Topology
===============
<p align="center"><img src="https://i.imgur.com/jHsQVL5.png"/></p>

`Mining Controller`: The mining controller is a new piece of software intended to be run by the individual miners. It connects to a full
node via the `work protocol`, to the pool via the `pool protocol` and to the mining software via the legacy stratum protocol. It receives block
templates from the full node, adds the coinbase requested by the pool, and translates it into the stratum protocol to send off to the miner.
When it receives a share from the miner, it translates it into the pool protocol and sends it to the pool.

`Full Node`: Full nodes which wish to be used for mining are expected to implement the `work protocol`. This protocol offers a streaming RPC which
pushes block templates to the controller whenever new work is available. The reason the mining controller is implemented in separate software rather 
than by the full node is to allow the controller to be run on a separate machine from the node and to allow the miner to connect to someone else's
full node if they wish to do so.

`Pool`: In this model the pool will send its mining address(s) to the controller when it first connects. Whenever a share is found the controller sends 
the block header, coinbase (containing the payout addresses requested by the pool) and the merkle hashes linking the coinbase to the header to the pool. 
The pool validates and tracks the shares and manages splitting the payout. It does not involve itself in transaction selection as that is the domain
of the individual miners. While the similest form for this new type of mining pool is a centralized pool which collects and distributes the reward, the
pool software could also take the form of something resembling p2pool which distributes the reward in a decentralized manner.

`Mining Software`: The mining software could be upgraded to handle the work and pool protocols and eliminate the need for a separate controller, 
but the quickest path to getting this deployed is for the controller to behave as a shim between the legacy stratum protocol and the new protocols. 
This means the mining software does not need to be patched to be compatible with the new mining network.

## References
[1] [BetterHash Mining Protocol](https://github.com/TheBlueMatt/bips/blob/betterhash/bip-XXXX.mediawiki#Abstract)
