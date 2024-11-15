---
title: Polkadot.js
description: Learn how to use the Polkadot.js API to interact with a Elysium node to get chain data and send transactions (extrinsics) via the Substrate side of Elysium.
---

# How to use the Polkadot.js API

The [Polkadot.js API](https://polkadot.js.org/docs/api/) library allows application developers to query a Elysium node
and interact with the node's Polkadot or Substrate interfaces using JavaScript. Here you will find an overview of the
available functionalities and some commonly used code examples to get you started on interacting with Elysium networks
using the Polkadot.js API library.

## What is Polkadot.js?

[Polkadot.js](https://wiki.polkadot.network/docs/polkadotjs) is a collection of tools that allow you to interact with
Polkadot and its parachains, such as Elysium. The Polkadot.js API is one component of the Polkadot.js collection and is
a JavaScript API that allows you to interface with a Elysium node to read and write data to the network.

You can use the Polkadot.js API to query on-chain data and send extrinsics from the Substrate side of Elysium. You can
query Elysium's runtime constants, chain state, events, transaction (extrinsic) data, and more.

## Checking Prerequisites

Installing and using Polkadot.js API library requires Node.js to be installed.

You need to install `Node.js` (for this example, you can use v16.x) and the npm package manager. You can download
directly
from Node.js or in your terminal:

### Ubuntu

```
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -

sudo apt install -y nodejs
```

### MacOs

```
# You can use homebrew (https://docs.brew.sh/Installation)
brew install node

# Or you can use nvm (https://github.com/nvm-sh/nvm)
nvm install node
```

You can verify that everything is installed correctly by querying the version for each package:

``` javascript
node -v
```

``` javascript
npm -v
```

### Install Polkadot.js API

First, you need to install the Polkadot.js API library for your project through a package manager such as `yarn`.
Install it in your project directory with the following command:

```
yarn add @polkadot/api
```

## Create an API Provider Instance

Similar to [Ethereum API libraries](/docs/category/libraries), you must first instantiate an API instance of the
Polkadot.js API. Create the `WsProvider` using the websocket endpoint of the Elysium network you wish to interact with.

To test out the examples in this guide on Elysium, you will need to have your own endpoint and API key,
which you can get from one of the supported [Endpoint Providers](/docs/network-endpoints).

```javascript
// Import
import {ApiPromise, WsProvider} from '@polkadot/api';

// Construct API provider
const wsProvider = new WsProvider('{{ networks.elysium.wss_url }}');
const api = await ApiPromise.create({provider: wsProvider});
```

### Metadata and Dynamic API Decoration

Before diving into the details of performing different tasks via the Polkadot.js API library, it's useful to understand
some of the basic workings of the library.

When the Polkadot.js API connects to a node, one of the first things it does is retrieve the metadata and decorate the
API based on the metadata information. The metadata effectively provides data in the form
of `api.<type>.<module>.<section>` that fits into one of the following `<type>` categories: `consts`, `query` and `tx`.

And therefore, none of the information contained in the `api.{consts, query, tx}.<module>.<method>` endpoints are
hard-coded in the API. This allows parachains like Elysium to have custom endpoints through its pallets that can be
directly accessed via the Polkadot.js API library.

## Query On-Chain Data on Elysium

In this section, you will learn how to query for on-chain information using the Polkadot.js API library.

### Elysium Chain State Queries

This category of queries retrieves information related to the current state of the chain. These endpoints are generally
of the form, `api.query.<module>.<method>`, where the module and method decorations are generated through metadata. You
can see a list of all available endpoints by examining the `api.query` object, via `console.log(api.query)` or
otherwise.

Here is a code sample for retrieving basic account information given its address:

```javascript
// Initialize the API provider as in the previous section
...

// Define wallet address
const addr = 'Elysium-WALLET-ADDRESS-HERE';

// Retrieve the last timestamp
const now = await api.query.timestamp.now();

// Retrieve the account balance & current nonce via the system module
const {nonce, data: balance} = await api.query.system.account(addr);

// Retrieve the given account's next index/nonce, taking txs in the pool into account
const nextNonce = await api.rpc.system.accountNextIndex(addr);

console.log(`${now}: balance of ${balance.free} and a current nonce of ${nonce} and next nonce of ${nextNonce}`);

// Disconnect the API
api.disconnect();
```

### Elysium RPC Queries

The RPC calls provide the backbone for the transmission of data to and from the node. This means that all API endpoints
such as `api.query`, `api.tx` or `api.derive` just wrap RPC calls, providing information in the encoded format as
expected by the node.

The `api.rpc` interface follows the a similar format to `api.query`, for instance:

```javascript
// Initialize the API provider as in the previous section
...

// Retrieve the chain name
const chain = await api.rpc.system.chain();

// Retrieve the latest header
const lastHeader = await api.rpc.chain.getHeader();

// Log the information
console.log(`${chain}: last block #${lastHeader.number} has hash ${lastHeader.hash}`);

// Disconnect the API
api.disconnect();
```

### Query Subscriptions

The RPCs lend themselves to using subscriptions. You can adapt the previous example to start using subscriptions to
listen to new blocks.

```javascript
// Initialize the API provider as in the previous section
...

// Retrieve the chain name
const chain = await api.rpc.system.chain();

// Subscribe to the new headers
await api.rpc.chain.subscribeNewHeads((lastHeader) => {
    console.log(`${chain}: last block #${lastHeader.number} has hash ${lastHeader.hash}`);
});

// Disconnect the API
api.disconnect();
```

The general pattern for `api.rpc.subscribe*` functions is to pass a callback into the subscription function, and this
will be triggered on each new entry as they are imported.

Other calls under `api.query.*` can be modified in a similar fashion to use subscription, including calls that have
parameters. Here is an example of how to subscribe to balance changes in an account:

```javascript
// Initialize the API provider as in the previous section
...

// Define wallet address
const addr = 'Elysium-WALLET-ADDRESS-HERE';

// Subscribe to balance changes for a specified account
await api.query.system.account(addr, ({nonce, data: balance}) => {
    console.log(`free balance is ${balance.free} with ${balance.reserved} reserved and a nonce of ${nonce}`);
});

// Disconnect the API
api.disconnect();
```

## Create a Keyring for a Elysium Account

The Keyring object is used for maintaining key pairs, and the signing of any data, whether it's a transfer, a message,
or a contract interaction.

### Create a Keyring Instance

You can create an instance by just creating an instance of the Keyring class, and specifying the default type of wallet
address used. For Elysium networks, the default wallet type should be `ethereum`.

```javascript
// Import the keyring as required
import Keyring from '@polkadot/keyring';

// Create a keyring instance
const keyring = new Keyring({type: 'ethereum'});
```

### Add an Account to a Keyring

There are a number of ways to add an account to the keyring instance, including from the mnemonic phrase and from the
shortform private key. The following sample code will provide some examples:

```javascript
// Import the required packages
import Keyring from "@polkadot/keyring";
import {u8aToHex} from "@polkadot/util";
import {mnemonicToLegacySeed, hdEthereum} from "@polkadot/util-crypto";

// Import Ethereum account from mnemonic
const keyringECDSA = new Keyring({type: "ethereum"});
const mnemonic = "MNEMONIC-HERE";
const privateKeyInput = 'PK-HERE';

// Define index of the derivation path and the derivation path
const index = 0;
const ethDerPath = "m/44'/60'/0'/0/" + index;
console.log(`Mnemonic: ${mnemonic}`);
console.log(`--------------------------\n`);

// Extract Ethereum address from mnemonic
const newPairEth = keyringECDSA.addFromUri(`${mnemonic}/${ethDerPath}`);
console.log(`Ethereum Derivation Path: ${ethDerPath}`);
console.log(`Derived Ethereum Address from Mnemonic: ${newPairEth.address}`);

// Extract private key from mnemonic
const privateKey = u8aToHex(
    hdEthereum(mnemonicToLegacySeed(mnemonic, "", false, 64), ethDerPath)
        .secretKey
);
console.log(`Derived Private Key from Mnemonic: ${privateKey}`);
console.log(`--------------------------\n`);

// Extract address from private key
const otherPair = keyringECDSA.addFromUri(privateKeyInput);
console.log(`Derived Address from provided Private Key: ${otherPair.address}`);

```

## Send Transactions on Elysium via the Polkadot.js API

Transaction endpoints are exposed, as determined by the metadata, on the `api.tx` endpoint. These allow you to submit
transactions for inclusion in blocks, be it transfers, deploying contracts, interacting with pallets, or anything else
Elysium supports.

### Send a Transaction

Here is an example of sending a basic transaction. This code sample will also retrieve the encoded calldata of the
transaction as well as the transaction hash after submitting.

```javascript
// Initialize the API provider as in the previous section
...

// Initialize the keyring instance as in the previous section
...

// Initialize wallet key pairs
const alice = keyring.addFromUri('ALICE-ACCOUNT-PRIVATE-KEY');
const bob = 'BOB-ACCOUNT-PUBLIC-KEY';

// Form the transaction
const tx = await api.tx.balances
    .transfer(bob, 12345n)

// Retrieve the encoded calldata of the transaction
const encodedCalldata = tx.method.toHex()
console.log(encodedCallData)

// Sign and send the transaction
const txHash = await tx
    .signAndSend(alice);

// Show the transaction hash
console.log(`Submitted with hash ${txHash}`);

// Disconnect the API
api.disconnect();
```

Note that the `signAndSend` function can also accept optional parameters, such as the `nonce`. For
example, `signAndSend(alice, { nonce: aliceNonce })`.

### Transaction Events

Any transaction will emit events, as a bare minimum this will always be either a `system.ExtrinsicSuccess`
or `system.ExtrinsicFailed` event for the specific transaction. These provide the overall execution result for the
transaction, i.e. execution has succeeded or failed.

Depending on the transaction sent, some other events may however be emitted, for instance for a balance transfer event,
this could include one or more `balance.Transfer` events.

### Batch Transactions

The Polkadot.js API allows transactions to be batch processed via the `api.tx.utility.batch` method. The batched
transactions are processed sequentially from a single sender. The transaction fee can be estimated using
the `paymentInfo` helper method. The following example makes a couple of transfers and also uses
the `api.tx.parachainStaking` module to schedule a request to decrease the bond of a specific collator candidate:

```javascript
// Initialize the API provider as in the previous section
...

// Initialize the keyring instance as in the previous section
...

// Initialize wallet key pairs as in the previous section
...

// Construct a list of transactions to batch
const collator = 'COLLATOR-ACCOUNT-PUBLIC-KEY';
const txs = [
    api.tx.balances.transfer(bob, 12345n),
    api.tx.balances.transfer(charlie, 12345n),
    api.tx.parachainStaking.scheduleDelegatorBondLess(collator, 12345n)
];

// Estimate the fees as RuntimeDispatchInfo, using the signer (either
// address or locked/unlocked keypair) 
const info = await api.tx.utility
    .batch(txs)
    .paymentInfo(alice);

console.log(`estimated fees: ${info}`);

// Construct the batch and send the transactions
api.tx.utility
    .batch(txs)
    .signAndSend(alice, ({status}) => {
        if (status.isInBlock) {
            console.log(`included in ${status.asInBlock}`);
        }
    });

// Disconnect the API
api.disconnect();
```

> **_NOTE:_**
> You can check out all of the available functions for the `parachainStaking` module by
> adding `console.log(api.tx.parachainStaking);` to your code.

## Substrate and Custom JSON-RPC Endpoints

RPCs are exposed as a method on a specific module. This means that once available, you can call any RPC
via `api.rpc.<module>.<method>(...params[])`. This also works for accessing Ethereum RPCs using the Polkadot.js API, in
the form of `polkadotApi.rpc.eth.*`.

Some of the methods availabe through the Polkadot.js API interface are also available as JSON-RPC endpoints on Elysium
nodes. This section will provide some examples; you can check for a list of exposed RPC endpoints by
calling `api.rpc.rpc.methods()` or the `rpc_methods` endpoint listed below.

- **[`methods()`](https://polkadot.js.org/docs/substrate/rpc/#methods-rpcmethods)**
    - **Interface** -  `api.rpc.rpc.methods`
    - **JSON-RPC** - `rpc_methods`
    - **Returns** - The list of RPC methods that are exposed by the node

      ```bash
        curl --location --request POST 'https://rpc.elysiumchain.tech' \
        --header 'Content-Type: application/json' \
        --data-raw '{
          "jsonrpc":"2.0",
          "id":1,
          "method":"rpc_methods",
          "params": []
        }'
      ```

- **[`getBlock(hash?: BlockHash)`](https://polkadot.js.org/docs/substrate/rpc/#getblockhash-blockhash-signedblock)**
    - **Interface** - `api.rpc.chain.getBlock`
    - **JSON-RPC** - `chain_getBlock`
    - **Returns** - The header and body of a block as specified by the block hash parameter

      ```bash
        curl --location --request POST 'https://rpc.elysiumchain.tech' \
        --header 'Content-Type: application/json' \
        --data-raw '{
          "jsonrpc":"2.0",
          "id":1,
          "method":"chain_getBlock",
          "params": ["0x870ad0935a27ed8684048860ffb341d469e091abc2518ea109b4d26b8c88dd96"]
        }'
      ```

- **[`getFinalizedHead()`](https://polkadot.js.org/docs/substrate/rpc/#getfinalizedhead-blockhash)**
    - **Interface** `api.rpc.chain.getFinalizedHead`
    - **JSON-RPC** `chain_getFinalizedHead`
    - **Returns** The block hash of the last finalized block in the canonical chain

      ```bash
        curl --location --request POST '{{ networks.elysium.rpc_url }}' \
        --header 'Content-Type: application/json' \
        --data-raw '{
          "jsonrpc":"2.0",
          "id":1,
          "method":"chain_getHeader",
          "params": []
        }'
      ```

## Polkadot.js API Utility Functions

The Polkadot.js API also includes a number of utility libraries for computing commonly used cryptographic primitives and
hash functions.

The following example computes the deterministic transaction hash of a raw Ethereum legacy transaction by first
computing its RLP ([Recursive Length Prefix](https://eth.wiki/fundamentals/rlp)) encoding, then hashing the result with
keccak256.

```javascript
import {encode} from '@polkadot/util-rlp';
import {keccakAsHex} from '@polkadot/util-crypto';
import {numberToHex} from '@polkadot/util';

// Define the raw signed transaction
const txData = {
    nonce: numberToHex(1),
    gasPrice: numberToHex(21000000000),
    gasLimit: numberToHex(21000),
    to: '0xc390cC49a32736a58733Cf46bE42f734dD4f53cb',
    value: numberToHex(1000000000000000000),
    data: '',
    v: "0507",
    r: "0x5ab2f48bdc6752191440ce62088b9e42f20215ee4305403579aa2e1eba615ce8",
    s: "0x3b172e53874422756d48b449438407e5478c985680d4aaa39d762fe0d1a11683"
}

// Extract the values to an array
var txDataArray = Object.keys(txData)
    .map(function (key) {
        return txData[key];
    });

// Calculate the RLP encoded transaction
var encoded_tx = encode(txDataArray)

// Hash the encoded transaction using keccak256
console.log(keccakAsHex(encoded_tx))
```

You can check the respective [NPM repository page](https://www.npmjs.com/package/@polkadot/util-crypto/v/0.32.19) for a
list of available methods in the @polkadot/util-crypto library and their descriptions.

