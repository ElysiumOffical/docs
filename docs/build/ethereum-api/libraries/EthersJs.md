---
title: Ethers.js
description: Follow this tutorial to learn how to use the Ethereum EtherJS Library to send transactions and deploy Solidity smart contracts to Elysium.
---

# Ethers.js JavaScript Library

The [Ethers.js](https://docs.ethers.io/) library provides a set of tools to interact with Ethereum Nodes
with JavaScript, similar to Web3.js. Elysium has an Ethereum-like API available that is fully compatible with
Ethereum-style JSON RPC invocations. Therefore, developers can leverage this compatibility and use the Ethers.js library
to interact with a Elysium node as if they were doing so on Ethereum. You can read more about Ethers.js on
this [blog post](https://medium.com/l4-media/announcing-ethers-js-a-web3-alternative-6f134fdd06f3)

In this guide, you'll learn how to use the Ethers.js library to send a transaction and deploy a contract on Atlantis
Testnet. This guide can be adapted for [Elysium](/docs/network-endpoints)

## Checking Prerequisites

For the examples in this guide, you will need to have the following:

- An account with funds. You can get LAVA for testing on once every 24 hours
  from [Elysium Faucet](https://faucet.atlantischain.network/ )
- To test out the examples in this guide on Atlantis, you will need to have your own endpoint and API key,
  which you can get from one of the supported [Endpoint Providers](/docs/network-endpoints).

> **_NOTE:_**
> The examples in this guide assumes you have a macOS or Ubuntu 18.04-based environment and will need to be adapted
> accordingly for Windows.

## Create a JavaScript Project

To get started, you can create a directory to store all the files you'll be creating throughout this guide:

```sh
mkdir ethers-examples && cd ethers-examples
```

For this guide, you'll need to install the Ethers.js library and the Solidity compiler. To install both NPM packages,
you can run the following command:

```
npm install ethers solc@0.8.1
```

## Setting up the Ethers Provider

Throughout this guide, you'll be creating a bunch of scripts that provide different functionality such as sending a
transaction, deploying a contract, and interacting with a deployed contract. In most of these scripts you'll need to
create an [Ethers provider](https://docs.ethers.io/v6/api/providers/) to interact with the network.

To create a provider, you can take the following steps:

1. Import the `ethers` library
2. Define the `providerRPC` object, which can include the network configurations for any of the networks you want to
   send a transaction on. You'll include the `name`, `rpc`, and `chainId` for each network
3. Create the `provider` using the `ethers.JsonRpcProvider` method

    ```js
    // 1. Import ethers
    const ethers = require('ethers');

    // 2. Define network configurations
    const providerRPC = {
      elysium: {
        name: 'atlantis',
        rpc: '{{ networks.rpc_url }}', // Insert your RPC URL here
        chainId: {{networks.chain_id}}, // Insert you
      },
    };
    // 3. Create ethers provider
    const provider = new ethers.JsonRpcProvider(
      providerRPC.elysium.rpc, 
      {
        chainId: providerRPC.elysium.chainId,
        name: providerRPC.elysium.name,
      }
    );
    ```

## Send a Transaction

During this section, you'll be creating a couple of scripts. The first one will be to check the balances of your
accounts before trying to send a transaction. The second script will actually send the transaction.

You can also use the balance script to check the account balances after the transaction has been sent.

### Check Balances Script

You'll only need one file to check the balances of both addresses before and after the transaction is sent. To get
started, you can create a `balances.js` file by running:

```
touch balances.js
```

Next, you will create the script for this file and complete the following steps:

1. [Set up the Ethers provider](#setting-up-the-ethers-provider)
2. Define the `addressFrom` and `addressTo` variables
3. Create the asynchronous `balances` function which wraps the `provider.getBalance` method
4. Use the `provider.getBalance` function to fetch the balances for the `addressFrom` and `addressTo` addresses. You can
   also leverage the `ethers.formatEther` function to transform the balance into a more readable number in ETH
5. Lastly, run the `balances` function

```js
// 1. Add the Ethers provider logic here:
// {...}

// 2. Create address variables
const addressFrom = 'ADDRESS-FROM-HERE';
const addressTo = 'ADDRESS-TO-HERE';

// 3. Create balances function
const balances = async () => {
    // 4. Fetch balances
    const balanceFrom = ethers.formatEther(await provider.getBalance(addressFrom));
    const balanceTo = ethers.formatEther(await provider.getBalance(addressTo));

    console.log(`The balance of ${addressFrom} is: ${balanceFrom} ETH`);
    console.log(`The balance of ${addressTo} is: ${balanceTo} ETH`);
};

// 5. Call the balances function
balances();
```

To run the script and fetch the account balances, you can run the following command:

```
node balances.js
```

If successful, the balances for the origin and receiving address will be displayed in your terminal in ETH.

### Send Transaction Script

You'll only need one file for executing a transaction between accounts. For this example, you'll be transferring 1 LAVA
from an origin address (from which you hold the private key) to another address. To get started, you can create
a `transaction.js` file by running:

```
touch transaction.js
```

Next, you will create the script for this file and complete the following steps:

1. [Set up the Ethers provider](#setting-up-the-ethers-provider)
2. Define the `privateKey` and the `addressTo` variables. The private key is required to create a wallet instance. **
   Note: This is for example purposes only. Never store your private keys in a JavaScript file**
3. Create a wallet using the `privateKey` and `provider` from the previous steps. The wallet instance is used to sign
   transactions
4. Create the asynchronous `send` function which wraps the transaction object and the `wallet.sendTransaction` method
5. Create the transaction object which only requires the recipient's address and the amount to send. Note
   that `ethers.parseEther` can be used, which handles the necessary unit conversions from Ether to Wei - similar to
   using `ethers.parseUnits(value, 'ether')`
6. Send the transaction using the `wallet.sendTransaction` method and then use `await` to wait until the transaction is
   processed and the transaction receipt is returned
7. Lastly, run the `send` function

```js
// 1. Add the Ethers provider logic here:
// {...}

// 2. Create account variables
const account_from = {
    privateKey: 'YOUR-PRIVATE-KEY-HERE',
};
const addressTo = 'ADDRESS-TO-HERE';

// 3. Create wallet
let wallet = new ethers.Wallet(account_from.privateKey, provider);

// 4. Create send function
const send = async () => {
    console.log(`Attempting to send transaction from ${wallet.address} to ${addressTo}`);

    // 5. Create tx object
    const tx = {
        to: addressTo,
        value: ethers.parseEther('1'),
    };

    // 6. Sign and send tx - wait for receipt
    const createReceipt = await wallet.sendTransaction(tx);
    await createReceipt.wait();
    console.log(`Transaction successful with hash: ${createReceipt.hash}`);
};

// 7. Call the send function
send();
```

To run the script, you can run the following command in your terminal:

```
node transaction.js
```

If the transaction was successful, in your terminal you'll see the transaction hash has been printed out.

You can also use the `balances.js` script to check that the balances for the origin and receiving accounts have changed.

## Deploy a Contract

The contract you'll be compiling and deploying in the next couple of sections is a simple incrementer contract,
arbitrarily named Incrementer.sol. You can get started by creating a file for the contract:

```
touch Incrementer.sol
 ```

Next you can add the Solidity code to the file:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract Incrementer {
    uint256 public number;

    constructor(uint256 _initialNumber) {
        number = _initialNumber;
    }

    function increment(uint256 _value) public {
        number = number + _value;
    }

    function reset() public {
        number = 0;
    }
}
```

The `constructor` function, which runs when the contract is deployed, sets the initial value of the number variable
stored
on-chain (default is 0). The `increment` function adds the `_value` provided to the current number, but a transaction
needs
to be sent, which modifies the stored data. Lastly, the reset function resets the stored value to zero.

### Compile Contract Script

In this section, you'll create a script that uses the Solidity compiler to output the bytecode and interface (ABI) for
the Incrementer.sol contract. To get started, you can create a compile.js file by running:

```
touch compile.js
```

Next, you will create the script for this file and complete the following steps:

1. Import the `fs` and `solc` packages
2. Using the `fs.readFileSync` function, you'll read and save the file contents of `Incrementer.sol` to `source`
3. Build the `input` object for the Solidity compiler by specifying the `language`, `sources`, and `settings` to be used
4. Using the `input` object, you can compile the contract using `solc.compile`
5. Extract the compiled contract file and export it to be used in the deployment script

```js
// 1. Import packages
const fs = require('fs');
const solc = require('solc');

// 2. Get path and load contract
const source = fs.readFileSync('Incrementer.sol', 'utf8');

// 3. Create input object
const input = {
    language: 'Solidity',
    sources: {
        'Incrementer.sol': {
            content: source,
        },
    },
    settings: {
        outputSelection: {
            '*': {
                '*': ['*'],
            },
        },
    },
};
// 4. Compile the contract
const tempFile = JSON.parse(solc.compile(JSON.stringify(input)));
const contractFile = tempFile.contracts['Incrementer.sol']['Incrementer'];

// 5. Export contract data
module.exports = contractFile;
```

### Deploy Contract Script

With the script for compiling the `Incrementer.sol` contract in place, you can then use the results to send a signed
transaction that deploys it. To do so, you can create a file for the deployment script called `deploy.js`:

```
touch deploy.js
```

Next, you will create the script for this file and complete the following steps:

1. Import the contract file from `compile.js`
2. [Set up the Ethers provider](#setting-up-the-ethers-provider)
3. Define the `privateKey` for the origin account. The private key is required to create a wallet instance. **Note: This
   is for example purposes only. Never store your private keys in a JavaScript file**
4. Save the `bytecode` and `abi` for the compiled contract
5. Create a wallet using the `privateKey` and `provider` from the previous steps. The wallet instance is used to sign
   transactions
6. Create a contract instance with signer using the `ethers.ContractFactory` function, providing the `abi`, `bytecode`,
   and `wallet` as parameters
7. Create the asynchronous `deploy` function that will be used to deploy the contract
8. Within the `deploy` function, use the `incrementer` contract instance to call `deploy` and pass in the initial value.
   For this example, you can set the initial value to `5`. This will send the transaction for contract deployment. To
   wait for a transaction receipt you can use the `deployed` method of the contract deployment transaction
9. Lastly, run the `deploy` function

```js
// 1. Import the contract file
const contractFile = require('./compile');

// 2. Add the Ethers provider logic here:
// {...}

// 3. Create account variables
const account_from = {
    privateKey: 'YOUR-PRIVATE-KEY-HERE',
};

// 4. Save the bytecode and ABI
const bytecode = contractFile.evm.bytecode.object;
const abi = contractFile.abi;

// 5. Create wallet
let wallet = new ethers.Wallet(account_from.privateKey, provider);

// 6. Create contract instance with signer
const incrementer = new ethers.ContractFactory(abi, bytecode, wallet);

// 7. Create deploy function
const deploy = async () => {
    console.log(`Attempting to deploy from account: ${wallet.address}`);

    // 8. Send tx (initial value set to 5) and wait for receipt
    const contract = await incrementer.deploy(5);
    const txReceipt = await contract.deploymentTransaction().wait();

    console.log(`Contract deployed at address: ${txReceipt.contractAddress}`);
};

// 9. Call the deploy function
deploy();
```

To run the script, you can enter the following command into your terminal:

```
node deploy.js
```

If successful, the contract's address will be displayed in the terminal.

### Read Contract Data (Call Methods)

Call methods are the type of interaction that don't modify the contract's storage (change variables), meaning no
transaction needs to be sent. They simply read various storage variables of the deployed contract.

To get started, you can create a file and name it `get.js`:

```
touch get.js
```

Then you can take the following steps to create the script:

1. Import the `abi` from the `compile.js` file
2. [Set up the Ethers provider](#setting-up-the-ethers-provider)
3. Create the `contractAddress` variable using the address of the deployed contract
4. Create an instance of the contract using the `ethers.Contract` function and passing in the `contractAddress`, `abi`,
   and `provider`
5. Create the asynchronous `get` function
6. Use the contract instance to call one of the contract's methods and pass in any inputs if necessary. For this
   example, you will call the `number` method which doesn't require any inputs. You can use `await` which will return
   the value requested once the request promise resolves
7. Lastly, call the `get` function

```js
// 1. Import the ABI
const {abi} = require('./compile');

// 2. Add the Ethers provider logic here:
// {...}

// 3. Contract address variable
const contractAddress = 'CONTRACT-ADDRESS-HERE';

// 4. Create contract instance
const incrementer = new ethers.Contract(contractAddress, abi, provider);

// 5. Create get function
const get = async () => {
    console.log(`Making a call to contract at address: ${contractAddress}`);

    // 6. Call contract 
    const data = await incrementer.number();

    console.log(`The current number stored is: ${data}`);
};

// 7. Call get function
get();
```

To run the script, you can enter the following command in your terminal:

```
node get.js
```

If successful, the value will be displayed in the terminal.

### Interact with Contract (Send Methods)

Send methods are the type of interaction that modify the contract's storage (change variables), meaning a transaction
needs to be signed and sent. In this section, you'll create two scripts: one to increment and one to reset the
incrementer. To get started, you can create a file for each script and name them `increment.js` and `reset.js`:

```
touch increment.js reset.js
```

Open the `increment.js` file and take the following steps to create the script:

1. Import the `abi` from the `compile.js` file
2. [Set up the Ethers provider](#setting-up-the-ethers-provider)
3. Define the `privateKey` for the origin account, the `contractAddress` of the deployed contract, and the `_value` to
   increment by. The private key is required to create a wallet instance. **Note: This is for example purposes only.
   Never store your private keys in a JavaScript file**
4. Create a wallet using the `privateKey` and `provider` from the previous steps. The wallet instance is used to sign
   transactions
5. Create an instance of the contract using the `ethers.Contract` function and passing in the `contractAddress`, `abi`,
   and `provider`
6. Create the asynchronous `increment` function
7. Use the contract instance to call one of the contract's methods and pass in any inputs if necessary. For this
   example, you will call the `increment` method which requires the value to increment by as an input. You can
   use `await` which will return the value requested once the request promise resolves
8. Lastly, call the `increment` function

```js
// 1. Import the contract ABI
const {abi} = require('./compile');

// 2. Add the Ethers provider logic here:
// {...}

// 3. Create variables
const account_from = {
    privateKey: 'YOUR-PRIVATE-KEY-HERE',
};
const contractAddress = 'CONTRACT-ADDRESS-HERE';
const _value = 3;

// 4. Create wallet
let wallet = new ethers.Wallet(account_from.privateKey, provider);

// 5. Create contract instance with signer
const incrementer = new ethers.Contract(contractAddress, abi, wallet);

// 6. Create increment function
const increment = async () => {
    console.log(
        `Calling the increment by ${_value} function in contract at address: ${contractAddress}`
    );

    // 7. Sign and send tx and wait for receipt
    const createReceipt = await incrementer.increment(_value);
    await createReceipt.wait();

    console.log(`Tx successful with hash: ${createReceipt.hash}`);
};

// 8. Call the increment function
increment();
```

To run the script, you can enter the following command in your terminal:

```
node increment.js
```

If successful, the transaction hash will be displayed in the terminal. You can use the `get.js` script alongside
the `increment.js` script to make sure that value is changing as expected.

Next you can open the `reset.js` file and take the following steps to create the script:

1. Import the `abi` from the `compile.js` file
2. [Set up the Ethers provider](#setting-up-the-ethers-provider)
3. Define the `privateKey` for the origin account and the `contractAddress` of the deployed contract. The private key is
   required to create a wallet instance. **Note: This is for example purposes only. Never store your private keys in a
   JavaScript file**
4. Create a wallet using the `privateKey` and `provider` from the previous steps. The wallet instance is used to sign
   transactions
5. Create an instance of the contract using the `ethers.Contract` function and passing in the `contractAddress`, `abi`,
   and `provider`
6. Create the asynchronous `reset` function
7. Use the contract instance to call one of the contract's methods and pass in any inputs if necessary. For this
   example, you will call the `reset` method which doesn't require any inputs. You can use `await` which will return the
   value requested once the request promise resolves
8. Lastly, call the `reset` function

```js
// 1. Import the contract ABI
const {abi} = require('./compile');

// 2. Add the Ethers provider logic here:
// {...}

// 3. Create variables
const account_from = {
    privateKey: 'YOUR-PRIVATE-KEY-HERE',
};
const contractAddress = 'CONTRACT-ADDRESS-HERE';

// 4. Create wallet
let wallet = new ethers.Wallet(account_from.privateKey, provider);

// 5. Create contract instance with signer
const incrementer = new ethers.Contract(contractAddress, abi, wallet);

// 6. Create reset function
const reset = async () => {
    console.log(`Calling the reset function in contract at address: ${contractAddress}`);

    // 7. sign and send tx and wait for receipt
    const createReceipt = await incrementer.reset();
    await createReceipt.wait();

    console.log(`Tx successful with hash: ${createReceipt.hash}`);
};

// 8. Call the reset function
reset();
```

To run the script, you can enter the following command in your terminal:

```
node reset.js
```

If successful, the transaction hash will be displayed in the terminal. You can use the `get.js` script alongside
the `reset.js` script to make sure that value is changing as expected.