---
title: "Atomic cross chain swap between Hyperledger Fabric and Ethereum"
date: 2021-11-28
tags:
- blockchain
- hyperledger
- crosschain
---

https://www.youtube.com/watch?v=j_j2MiAxUvY&list=PL0MZ85B_96CEmmy0C6NF52ZCMNcY1Wryf

## Methods

### Wait until the transaction is completely settled

not practical and safe

### Hashlock

The transaction is partially complete

Settle once the sender publishes the key on the blockchain.

#### Cons

* hugo cannot refund if kevin does not response

### Timelock

* hugo made a partial transaction that the btc can be claimed by kevin within n number of blocks
* if the transaction is expired, hugo can claim back the fund

#### Cons
* the claims need to execute manually

#### HTLC (Hash Time Lock Contract)

Combine two locks: (and)

* claim within n number of blocks
* the key is revealed on the blockchain 

### Steps in a cross chain swap

Timestamp: 55:36

Example: btc -> eth swap from hugo to kevin

assume that Hugo has a key (secret that only Hugo will know) `1234` with hash `4567`

Hugo and Kevin will not make transactions directly

They will send the fund to the contract instead

THe contract will hold the eth and btc for Hugo and Kevin

There are 2 separate contracts
* one is in btc (let's say `Contract A`)
* the other is in eth (let's say `Contract B`)

* hugo sent btc to the `Contract A` in btc network

Transaction:

| Field | Value |
|---|---|
| From  | hugo |
| Value | 10btc |
| To | kevin |
| Hash | 4567 |

* Kevin sent eth to the `Contract B` in eth network

| Field | Value |
|---|---|
| From  | kevin |
| Value | 134eth |
| To | hugo |
| hash | 9876 |

* Hugo observed that kevin has funded eth to `Contract B`. then he will submit his key `1234` in btc network, btc send to kevin by the `Contract A` 
* same as kevin, he submits his key `9876` in eth network, eth sent to hugo by the `Contract B`

### Problem

* semi-trusted
  * we assume Hugo and Kevin can see each other on each chain
  * if the access right is changed during the transaction...let say hugo can revoke kevin on eth network..then the transaction fails (this happens on hyperledger as it can have access control but it is not possible on btc and eth)
