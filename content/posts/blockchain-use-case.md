---
title: "Blockchain use cases"
date: 2021-11-26
tags:
- use-case
- blockchain
- hyperledger
---

## Trust Over IP: Hyperledger Ursa, Indy, and Aires

https://www.youtube.com/watch?v=FfuhlF9ZYPM

### Notes
* prove the credential without CA
* trusted issuer (eg: gov) issues the identity token (eg: driving licence) -> send it to a crypto account (like sending btc to someone)
* on the blockchain, it shows a record that the account owns that licence. (like you can check your btc balance / NFS on chain explorer)
* anyone can request that you own that account (like an approval transaction from dapp). You can decide to share your data to what extent 
* send the data to requester and it can be verified as it is signed by issuer
* blockchain is used to prove the ownership of your identities
* data can be stored off chain, sign it and verify with issuer's account
* hacking personal accounts become useless -> not cost effective, you can hack a single person each time
* stealing / leading personal data is useless -> no proof, no one will accept

### Example
[british columbia](https://www2.gov.bc.ca/gov/content/home)

## Comprehensive review of 21 use cases of Hyperledger

https://youtu.be/VQTARQSuUFU?t=1440

### Use cases (some are still in proof of concept)

1. end to end tracking (supply chain, different parties can participate eg: bank insurance company consultant retail...)
2.identity management - eg: allow 3rd parties to access your data with requests, prevent fraud, credit checking, hash your passport/fingerprint so you can verify it is your identity
3. contract automation- like promotion in company (HR), consultant contract
4. voting
5. tokenisations - cross parties reward program(asiamiles), assets management (use in trading, convert different kinds of assets like house stock cash to token)
6. immutable record- law enforcement investigation, accounting, credit check, insurance claim, medical history, law...

## Hyperledger Fabric Business Use Cases

https://youtu.be/1ps4PJtFRfY?t=544

### IBM food trust
* safety
* sustainability
* cost optimization (inventory / transportation )
* tracking

### Tradelens -> (supply chain for trading)
* costs 1.8T / year -> potential saving ~10% costs

### IBM blockchain world wire
* world wire solution with blockchain
* use stellar protocol

## Singapore Exchange: Clearing Financial Transactions on Amazon Managed Blockchain

https://www.youtube.com/watch?v=nUre2ELySdo

blockchain is used to handle the payment and settlement of securities

the method they used called DVP (delivery versus payment) = a way to ensure cash and securities can simultaneously exchange 

client can be 
* buyer = buy securities with eth
* seller = sell securities with eth

In the architecture,
* hyperledger = securities
* eth client = cash

Transaction will come across with both ledgers

arbitrator and observer with receive activities on chain

* arbitrator = mediator = handle dispute
* observer = regulator = do regulatory report

### Pros with AWS Managed Blockchain

* no need to build the ledger by themself
* focus on the smart contract
* a single blockchain network can effectively work across multiple parties

### An Overview on Blockchain Services from AWS

https://www.youtube.com/watch?v=WAIOBeQA2QQ

#### Pros of blockchain network
* ledger database (immutable transaction)
* consensus
* decentralisation
* smart contract
* ease for audit 

#### Hyperledger fabric VS ETH
* authentication
* access control
* easy to add user
* easy to duel with settlement (bank)
* faster than centialization
* private ledger

### use case 
* trading
* supply case

## Enhanced Transparency and Frictionless Supply Chain with Blockchain

https://www.youtube.com/watch?v=2NqbUb4rT88&list=WL&index=46

B2B Business

buy and sell order between Bottlers

* visibility cross supply chain
* reduce manual process and communicate between sellers and buyers
* dispute resolution (smark contract) -> I think we can use TimeLock to do partial transaction
* easy to scale
* no need to build a different api for each system (mesh network n(n-1)/2)
