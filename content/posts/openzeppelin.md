---
title: "OpenZeppelin"
date: 2021-12-01
tags:
- smart-contract
- blockchain
---

## Upgradable contract

https://www.youtube.com/watch?v=kWUDTZhxKZI&list=WL&index=19

### proxy pattern

user interact with the proxy contract

the proxy will point to another smart contract

When upgrading the contract, the admin of the proxy contract points the proxy to another smart contract
storage

### Implementation

* Transparent
* UUPS
