---
title: Accounts
description: Guide to Stacks 2.0 accounts
---

## Introduction

Stacks 2.0 accounts are entities that own assets, like Stacks (STX) tokens. An account has an address, private key, nonce, and one or more asset balances.

If you want to jump right in to generate and query a new account, try this tutorial:

[@page-reference | inline]
| /stacks-blockchain/managing-accounts

-> The public-key signature system used for Stacks 2.0 accounts is [Ed25519](https://ed25519.cr.yp.to/).

Assets cannot leave an account without an action from the account owner. All changes to assets (and the balances of the account) require a corresponding transaction.

-> The [transaction type](/stacks-blockchain/transactions#types) doesn't need to be a token transfer - contract deploy and contract call transactions can change the balances of an account

## Creation

An account is generated from a 24-word mnemonic phrase. This is often referred to as the **seed phrase**. The seed phrase provides access to Stacks 2.0 accounts.

!> If the seed phrase is lost, access to the associated account cannot be restored. No person or organization, including Blockstack, can recover a lost seed phrase.

The easiest way to generate a new Stacks 2.0 account is to use the [Stacks 2.0 CLI](https://github.com/blockstack/cli-blockstack/tree/feature/stacks-2.0-tx):

```bash
# install CLI globally
npm install --global "https://github.com/blockstack/cli-blockstack#feature/stacks-2.0-tx"

# generate a new account and store details in a new file
# '-t' option makes this a testnet account
blockstack make_keychain -t > cli_keychain.json
```

`make_keychain` creates the following file:

```js
{
  "mnemonic": "aaa bbb ccc ddd ...",
  "keyInfo": {
    "address": "STJRM2AMVF90ER6G3RW1QTF85E3HZH37006D5ER1",
    "privateKey": "5a3f1f15245bb3fb...",
    "index": 0,
    "btcAddress": "biwSd6KTEvJcyX2R8oyfgj5REuLzczMYC1"
  }
}
```

| Field                | Description                                                                                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `mnemonic`           | A 24-word seed phrase used to access the account, generated using [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) with 256 bits of entropy |
| `keyInfo.address`    | Stacks address for the account                                                                                                                                     |
| `keyInfo.privateKey` | Private key for the account. Required for [token transfers](/stacks-blockchain/transactions#stacks-token-transfer) and often referred to as `senderKey`            |
| `keyInfo.index`      | Nonce for the account, starting at 0                                                                                                                               |
| `keyInfo.btcAddress` | Corresponding BTC address for the account. A construct from the previous blockchain (Stacks 1.0) and currently unused.                                             |

-> `btcAddress` was used in the old, Stacks 1.0, blockchain. Stacks 1.0 accounts would pay for occurring transaction fees with the corresponding BTC account. This is **not** the case for Stacks 2.0 accounts anymore and the field is a leftover

Note that a new account automatically exists for each new private key. There is no need to manually instantiate an account on the Stacks 2.0 blockchain.

-> Addresses are created by generating the [RIPEMD-160 hash](https://en.wikipedia.org/wiki/RIPEMD#RIPEMD-160_hashes) of the [SHA256](https://en.bitcoinwiki.org/wiki/SHA-256) of the public key. BTC addresses are encoded with [Base58Check](https://en.bitcoin.it/wiki/Base58Check_encoding). For Stacks addresses, [c32check](https://github.com/blockstack/c32check) is used. Deriving an address from a public key can be done without internet access, for instance using the c32check `c32addressDecode` method.

Alternatively to the CLI creation, the [Stacks Transactions JS](https://github.com/blockstack/stacks-transactions-js) library can be used:

```js
import {
  makeRandomPrivKey,
  privateKeyToString,
  getAddressFromPrivateKey,
  TransactionVersion,
} from '@blockstack/stacks-transactions';

const privateKey = makeRandomPrivKey();

// Get public key from private
const publicKey = getPublicKey(privateKey);

const stacksAddress = getAddressFromPrivateKey(
  privateKeyToString(privateKey),
  TransactionVersion.Testnet // remove for Mainnet addresses
);
```

## Querying

### Get Stacks (STX) balance and nonce

STX balance and nonce can be obtained through the [`GET /v2/accounts/<stx_address>`](https://blockstack.github.io/stacks-blockchain-api/#operation/get_account_info) endpoint:

```bash
curl 'https://stacks-node-api.blockstack.org/v2/accounts/<stx_address>'
```

Sample response:

```js
{
    "balance": "0x0000000000000000002386f26f3f40ec",
    "nonce": 17
}
```

-> The balance string represents an unsigned 128-bit integer (big-endian) in hex encoding

### Get all token balances

All token balances can be obtained through the [`GET /extended/v1/address/<stx_address>/balances`](https://blockstack.github.io/stacks-blockchain-api/#operation/get_account_balance) endpoint:

```bash
curl 'https://stacks-node-api.blockstack.org/extended/v1/address/<stx_address>/balances'
```

Sample response:

```js
{
    "stx": {
        "balance": "0",
        "total_sent": "0",
        "total_received": "0"
    },
    "fungible_tokens": {},
    "non_fungible_tokens": {}
}
```

-> Stacks accounts cannot hold bitcoins. The best way to obtain corresponding BTC balances is to derive the BTC address from the Stacks address (using [`c32check`](https://github.com/blockstack/c32check#c32tob58-b58toc32)) and query the Bitcoin network.

### Get all asset events

All asset events associated with the account can be obtained through the [`GET /extended/v1/address/<stx_address>/assets`](https://blockstack.github.io/stacks-blockchain-api/#operation/get_account_balance) endpoint:

```bash
curl 'https://stacks-node-api.blockstack.org/extended/v1/address/<stx_address>/assets'
```

Sample response:

```js
{
    "limit": 20,
    "offset": 0,
    "total": 0,
    "results": [
        {
            "event_index": 5,
            "event_type": "non_fungible_token_asset",
            "asset": {
                "asset_event_type": "transfer",
                "asset_id": "ST2W14YX9SFVDB1ZGHSH40CX1YQAP9XKRAYSSVYAG.hello_world::hello-nft",
                "sender": "SZ2J6ZY48GV1EZ5V2V5RB9MP66SW86PYKKQ9H6DPR",
                "recipient": "SM2J6ZY48GV1EZ5V2V5RB9MP66SW86PYKKQVX8X0G",
                "value": {
                    "hex": "0x0100000000000000000000000000000001",
                    "repr": "u1"
                }
            }
        },
        {
            "event_index": 3,
            "event_type": "fungible_token_asset",
            "asset": {
                "asset_event_type": "mint",
                "asset_id": "ST2W14YX9SFVDB1ZGHSH40CX1YQAP9XKRAYSSVYAG.hello_world::novel-token-19",
                "sender": "",
                "recipient": "SZ2J6ZY48GV1EZ5V2V5RB9MP66SW86PYKKQ9H6DPR",
                "amount": "12"
            }
        }
    ]
}
```

-> Read more about [pagination](/stacks-blockchain/transactions#pagination) to iterate through the entire result set of the asset events
