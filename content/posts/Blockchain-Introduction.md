---
title: "Blockchain Introduction"
date: 2022-01-22
tags:
- blockchain
---

# Resources

* [Hyperledger Fabric](https://www.youtube.com/watch?v=iTV89Tqfmgk&list=PLcjWRSA2O5d0os20SjN3Q_21PeVdpKHxy)
* [Solidity](https://www.youtube.com/playlist?list=PLcjWRSA2O5d0CrOBe2i1oLThyQhFWo19H)

# Notes

## what is blockchain 

![blockchain](https://www.euromoney.com/learning/~/media/0E855B86EDF04F3C8EABAFC42917C8C6.png?la=en&hash=B67568B63CBB2C0C7311DE742F5B9E48E86DC8B9)

* The blockchain solution in IoT / Supply chain are changing "Mesh" network to a "Star Network
* each parties access and write to the "blockchain network". not more api integration between parties

<img src="https://images.saymedia-content.com/.image/t_share/MTc0NDgwMDY5MTgyMzY3MzY2/taking-back-the-internet-mesh-networks-are-more-reliable-and-lower-cost.gif" alt="Mest Network" style="max-width:400px;"/>

![Star Network](http://webpage.pace.edu/ms16182p/networking/star2.gif)

## How to use it

## How it work

* use cryptography methods to ensure the integrity of a public writable database
* proof of work, proof of stake, proff

## How to implement 

* solidity
* node.js
* truffle
* web3.js

## Any product (e.g crypto currency/ nft)

### Networks

* [Use Cases](/posts/blockchain-use-case)

#### Public

* [btc](https://www.blockchain.com/explorer)
* [eth](https://etherscan.io/)
* [bsc (binance smark contract)](https://bscscan.com/)

#### Private (have acl)

* each parties setup servers to join the blockchain network
* all parties hold the data but they can set access right to the data

* [Hyperledger Fabric](https://www.hyperledger.org/use/fabric)

### Decentralized Storage

* [IPFS](https://ipfs.io/)

### Staking

### CrowdSale

user -> sale contract -> nft contract

### Lottery

* [ashisherc/advanced-solidity-lottery-application](https://github.com/ashisherc/advanced-solidity-lottery-application)

### NFT

* [Auction](https://forum.openzeppelin.com/t/any-sample-of-auction-that-combines-with-erc721/6876) ([dutch](https://solidity-by-example.org/app/dutch-auction/) and [English](https://solidity-by-example.org/app/english-auction/))
* [mint](https://github.com/ProjectOpenSea/opensea-creatures)
* [CrowdSale](https://forum.openzeppelin.com/t/simple-erc20-crowdsale/4863)
* [ERC1155](https://docs.openzeppelin.com/contracts/3.x/erc1155)
* [LootBox](https://github.com/ProjectOpenSea/opensea-erc1155/tree/master/contracts)

* [opensea](https://opensea.io/)
* [scv.finance](https://scv.finance/market)
* [La Collection](https://lacollection.io/gallery)

### Swap / Farming

* ask people input proportion of coin into the pool, like eth:bnb. so people can trade between eth and bnb
* use Oracles for pricing...a smart contract which referencing different sources from other swap exchange & their coin pair pool size

* [pancakeswap(bsc)](https://pancakeswap.finance/swap)
* [uniswap(eth)](https://info.uniswap.org/)

## What is ECR20 721 1155

![ECR20 vs ECR721](http://maikotrindade.github.io/public/img/nonfungibletable.png)

* ECR20 = coin
* ECR721 = NFT
* [ECR1155](https://blog.enjincoin.io/erc-1155-the-crypto-item-standard-ac9cf1c5a226) = 1 contract with multi token, eg: game item

![ERC1155 Swap](https://miro.medium.com/max/503/1*XlKs23mB2BTzjMOHfCp9-w.png)

![ECR115 Batch](https://miro.medium.com/max/698/1*53Ovjtik7_YWB5yKFm9XsQ.png)

## What is smart contract

* [Code stored on blockchain](https://www.ibm.com/topics/smart-contracts)

# Demo

## NFT Contract

* [NFT Example](https://scv.finance/nft/bsc/0x85F0e02cb992aa1F9F47112F815F519EF1A59E2D/10000586756)

* [Contract Wizard](https://wizard.openzeppelin.com/)

Common Functions

* Mintable (increase supply)
* Burnable (reduce supply)
* Pausable (freeze the contract, for upgrade)
* Access control: Roles (multi ac, multi actions) / Ownable (single ac, all actions)
* Upgradeability (Transpart / UUPS) - proxy pattern
* Timelock
* Votes

### [NFT metadata](https://meta.polkamon.com/meta?id=10000586756)

```
{
  "boosterId": 10000000195586,
  "id": "10000586756",
  "txHash": "0xff318896b78fd77aadf19f94b7434d1a0ea5ffddfc88b6966d92d75c69f80dd1",
  "randomNumber": "0xc55ec3b26d0bdec23fd2f9e29ae34d54e0d457ef1d413b5c537b0383330575f0",
  "image": "https://assets.polkamon.com/images/Unimons_T08C02H06B04G00.jpg",
  "external_url": "https://polkamon.com/polkamon/T08C02H06B04G00",
  "description": "Mindful of a million things, the Unisheep are Polkamon creatures that can graze and gander the day away. Creating an environment to actively direct these creatures helps them focus on one particular topic.",
  "name": "Unisheep",
  "initialProbabilities": {
    "horn": 0.2,
    "color": 0.345,
    "background": 1,
    "glitter": 0.99,
    "type": 0.1847
  },
  "attributes": [
    {
      "trait_type": "Type",
      "value": "Unisheep"
    },
    {
      "trait_type": "Horn",
      "value": "Spiral Horn"
    },
    {
      "trait_type": "Color",
      "value": "Green"
    },
    {
      "trait_type": "Background",
      "value": "Mountain Range"
    },
    {
      "trait_type": "Opening Network",
      "value": "Binance Smart Chain"
    },
    {
      "trait_type": "Glitter",
      "value": "No"
    },
    {
      "trait_type": "Special",
      "value": "No"
    },
    {
      "display_type": "date",
      "trait_type": "Birthday",
      "value": 1628902374
    },
    {
      "display_type": "number",
      "trait_type": "Booster",
      "value": 10000000195586
    }
  ],
  "opening_network": "Binance Smart Chain",
  "background_color": "FFFFFF",
  "animation_url": "https://assets.polkamon.com/videos/Unimons_T08C02H06B04G00.mp4",
  ...
}
```

### [Read Contract](https://bscscan.com/token/0x85F0e02cb992aa1F9F47112F815F519EF1A59E2D?a=10000586756#readContract)

* Owner account: [0xaB111a3EaFE79b3110162d0e8b6FF1102ed25E2A](https://scv.finance/nft/user/0xaB111a3EaFE79b3110162d0e8b6FF1102ed25E2A?tab=in-wallet)
* token id: [10000586756](https://scv.finance/nft/bsc/0x85F0e02cb992aa1F9F47112F815F519EF1A59E2D/10000586756)

Functions:
* balanceOf (3)
* getApproved (4)
* Role: DEFAULT_ADMIN_ROLE / MINTER_ROLE (1,2)
* getRoleMember (6)
* ownerOf (11)

## [Write Contract](https://bscscan.com/token/0x85F0e02cb992aa1F9F47112F815F519EF1A59E2D?a=10000586756#writeContract)

Functions:
* burn (2)
* mint (4)
* grantRole (3)
* safeTranferFrom
* setTokenURI

## Wallet

* [wordphase - BIP39](https://iancoleman.io/bip39/)

### Connect from browser

* [PancakeSwap](https://pancakeswap.finance/swap)
* [Remix IDE](https://remix.ethereum.org/)
* [bscscan.com](https://bscscan.com/token/0x85F0e02cb992aa1F9F47112F815F519EF1A59E2D?a=10000586756#writeContract)

### Contract deployment

* [Code Example](https://github.com/hugotkk/Solidity-Course-Project/blob/main/deploy_lottery.js)
* will need a wallet and [Blockchain endpoint](https://infura.io/)

# References

* https://www.euromoney.com/learning/~/media/0E855B86EDF04F3C8EABAFC42917C8C6.png?la=en&hash=B67568B63CBB2C0C7311DE742F5B9E48E86DC8B9
