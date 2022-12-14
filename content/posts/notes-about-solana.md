---
title: "Notes about Solana"
date: 2022-12-14
tags:
- blockchain
- solana
- nft
---

# Useful links

## Explorer

* [solana explorer](https://explorer.solana.com/)

## NFT marketplace

* [solanart](https://solanart.io/)
* [solasea](https://solsea.io/)

## Good articles to understand the concept of solana

* [Solana NFT 101](https://twitter.com/pencilflip/status/1506310957588430861)
* [Solana’s Token Program, Explained](https://pencilflip.medium.com/solanas-token-program-explained-de0ddce29714)
* [Understanding Solana’s Mint Accounts and Token Accounts](https://medium.com/@jorge.londono_31005/understanding-solanas-mint-account-and-token-accounts-546c0590e8e)
* [How NFTs are represented in Solana](https://lorisleiva.com/owning-digital-assets-in-solana/how-nfts-are-represented-in-solana)

In the Solana world, data will be stored in different accounts. Accounts will be associated with each other by cross-referencing. The Solana program will be used to manipulate those accounts (create, delete, set link). Business logic will be introduced from a separated program. 

Common tasks eg: creating token / nft can be done with some universal solana programs ([spl Token program](https://spl.solana.com/token), [metaplex](https://docs.metaplex.com/)) on solana network. Developers don't need to write their own solana program. This makes things become more standard. People write their own business logic by extending the functions of those universal programs but won't start from zero.

# Notes

* [solfaucet](https://solfaucet.com/) can claim free SOL (devnet) for development
* Most of the people use [metaplex](https://docs.metaplex.com/) to create nft
* Metaplex's [Token Metadata Program](https://docs.metaplex.com/programs/token-metadata/) defined the [standard of a solana NFT](https://docs.metaplex.com/programs/token-metadata/changelog/v1.0)
* Metaplex's [candy machine](https://docs.metaplex.com/programs/candy-machine/) provided abilities to sell the nft eg: loot box, start/end date, whitelist
* [sugar](https://docs.metaplex.com/developer-tools/sugar/tutorials/my-first-candy-machine) is a cli tool to create candy machines. It simplified the task but I got a "Request header too large" error when uploading a large file (100MB)
* [Metaplex javascript sdk](https://github.com/metaplex-foundation/js#installation) can solve the issue of sugar but their [documentation](https://docs.metaplex.com/programs/candy-machine/managing-candy-machines) at this moment is a bit old (even on github README). The examples are still in V1 but sugar cli has already moved V2. The syntax of V2 is quite different from V1. Therefore, I have to refer to this [sdk doc](https://metaplex-foundation.github.io/js/classes/js.Metaplex.html#candyMachinesV2) for development.
* If "animation_url" is set to a mp4 / gif, it will playable inside the solana wallet (like phantom). I thought it was related to the "properties.files" field.
* "properties.files" should include all related resources. For examples, if the NFT have a mp4 "animation_url" and jpg "image", "properties.files" should include both
* When uploading files to arweave with metaplex sdk, it will not append `?ext=<extension>` to the URL. This caused the image and videos to fail to be shown correctly on the phantom wallet. The file extension should always be included like https://www.arweave.net/efgh1234?ext=mp4.
* this [calculator](https://55mcex7dtd5xf4c627v6hadwoq6lgw6jr4oeacqd5k2mazhunejq.arweave.net/71giX-OY-3LwXtfr44B2dDyzW8mPHEAKA-q0wGT0aRM) is useful to calculator how much do we need to pay for the persistent storage on arweave
* [spl token program cli](https://spl.solana.com/token#setup) is used to create token / nft but it provides the basic functions only. For instance, there is no metadata for the nft token created from the token program. Metaplex is kind of extension which added the metadata and venting machine functions on top of the token program
* [solana cli](https://docs.solana.com/cli/install-solana-cli-tools#linux) - will be used to create accounts / transfer / check balances. More use cases are in [here](https://docs.solana.com/wallet-guide/paper-wallet).
* When [create a solana account](https://docs.solana.com/wallet-guide/paper-wallet#seed-phrase-generation), it will print out 12-words seed phases and save the private key in `~/.config/solana/id.json`. Please save the seed phases. The private key can be derived from seed phases but we can't recover the seed phases from private keys.
* Solana-cli-created accounts can only be imported to the phantom wallet with a private key. The default derivation path of Phantom is different from solana-cli. Changing the derivation path is only supported when importing a hardware wallet to phantom. Therefore, there is no way to import the account from seed phases.
* To do that, we have to convert uint8array (id.json) to base58. This can be done by library like [base58-js](https://www.npmjs.com/package/base58-js/v/1.0.0)
* [web3.js](https://docs.solana.com/developing/clients/javascript-api#installation) - web3 library of solana but people usually use the [anchor's version](https://coral-xyz.github.io/anchor/ts/modules/web3.html) because most of the solana programs are developed from anchor
* [this](https://docs.solana.com/clusters) shows the public clusters in the solana network. I use [quicknode](https://www.quicknode.com) to create a dedicated solana endpoint instead.
* More examples can be found on [Solana Cookbook](https://solanacookbook.com/references/token.html), [Solana Web3 Demo](https://yihau.github.io/solana-web3-demo/), [Solana Development Guide](https://www.quicknode.com/guides/solana-development/)

