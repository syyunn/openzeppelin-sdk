---
id: interacting
title: Interacting with your contracts
---

This tutorial was [built on the previous one](first) where we started a new ZeppelinOS project and created (and upgraded!) a simple `Counter` contract in a local development network. We will now see how to interact with this contract from javascript code using [web3.js](https://web3js.readthedocs.io/en/1.0/). We will build a small script that will increase the counter and report back the updated value.

The full code for this tutorial can be found in the [`first-project` example](https://github.com/zeppelinos/zos/blob/v2.4.0/examples/first-project/src/index.js) in the ZeppelinOS github repository.

## Setup

We will be using [web3.js](https://web3js.readthedocs.io/en/1.0/) to interact with the blockchain from our code, which is the same library that ZeppelinOS uses under the hood. In the `my-project` folder we created earlier, run the following to install it:

```console
npm install web3@1.0.0-beta.37
```

> Note: We are using beta.37 version at the moment due to incompatibility issues with later versions, but expect to see these guides updated to 1.0.0 stable soon. Also, if you run into any issues installing `web3`, make sure you are using node 10 and not 12.

Keep in mind that there are many other javascript libraries available, and you can use whichever you like the most. Once a contract is deployed, you can interact with it through any library!

## Connecting to the network

Our first step will be to open a connection to the network. We will connect to the local development network we started on the previous tutorial. 

> Caution: By default, `ganache-cli` deletes all data when you stop it. If you stopped the ganache process from the previous tutorial, you will need to start a new one with `ganache-cli --deterministic` and run `zos create Counter` again.

Let's begin coding in a new `src/index.js` file, where we will be writing our javascript script. We will start with some boilerplate for writing async code and setting up a new `web3` object.

<!-- Code: We should provide a function to automatically parse `networks.js` and get the web3 instance preconfigured -->

```js
const Web3 = require('web3');

async function main() {
  // Set up web3 object, connected to the local development network
  const web3 = new Web3('http://localhost:8545');
}

main();
```

Here, we are initializing a new `web3` instance connecting to the local development network that is running on localhost port 8545. We can test if the connection works by asking something to the local node, such as the list of enabled accounts:

```js
// Set up web3 object, connected to the local development network
const web3 = new Web3('http://localhost:8545');

// Retrieve accounts from the local node
const accounts = await web3.eth.getAccounts();
console.log(accounts);
```

> Note: We will not be repeating the boilerplate code on every snippet, but make sure to always code _inside_ the `main` function we defined above.

Run the code above using `node`, and check that you are getting a list of available accounts in response. These should match the ones you saw when first starting the `ganache-cli` process.

```console
$ node src/index.js 
[ '0x90F8bf6A479f320ead074411a4B0e7944Ea8c9C1',
  '0xFFcf8FDEE72ac11b5c542428B35EEF5769C409f0',
  '0x22d491Bde2303f2f43325b2108D26f1eAbA1e32b',
  '0xE11BA2b4D45Eaed5996Cd0823791E0C93114882d',
  '0xd03ea8624C8C5987235048901fB614fDcA89b117',
  '0x95cED938F7991cd0dFcb48F0a06a40FA1aF46EBC',
  '0x3E5e9111Ae8eB78Fe1CC3bb8915d5D461F3Ef9A9',
  '0x28a8746e75304c0780E011BEd21C72cD78cd535E',
  '0xACa94ef8bD5ffEE41947b4585a84BdA5a3d3DA6E',
  '0x1dF62f291b2E969fB0849d99D9Ce41e2F137006e' ]
```

Great! We have our first code snippet getting data out of a blockchain. Let's start working with our contract now.

## Getting a contract instance

In order to interact with our deployed `Counter` contract, we will create a new [web3 contract instance](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html). This is a javascript object that represents our contract in the blockchain. 

To create this web3 contract, we need two things:
- The address where it is deployed
- The ABI of the contract, which is the definition of the contract's public functions

<!-- Code: We should provide both a command and a js function to easily retrieve the address from the network.json file. Same for the ABI, and for building a web3 contract altogether. -->

The address was returned by the ZeppelinOS CLI when we deployed the contract. As for the ABI, we can retrieve it from the compiled artifact in the `build/contracts` folder as shown below.

```js
// Set up web3 object, connected to the local development network
const web3 = new Web3('http://localhost:8545');

// Set up a web3 contract, representing our deployed Counter instance
const address = '0xCfEB869F69431e42cdB54A4F4f105C19C080A601';
const abi = require('../build/contracts/Counter.json').abi;
const counter = new web3.eth.Contract(abi, address);
```

> Note: Make sure to replace the `address` with the one you got when deploying the contract, which may be different to the one shown here.

We can now use this javascript object to interact with our contract. Let's see how!

## Calling the contract

Let's start by displaying the current value of the `Counter` contract. We will need to [call](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#methods-mymethod-call) into the `value()` public method of the contract, and [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) the response:

```js
// Call the value() function of the deployed Counter contract
const value = await counter.methods.value().call();
console.log(value);
```

Make sure everything is running smoothly by running the script again and checking the printed value:

```console
$ node src/index.js
11
```

> Note: If you restarted ganache between the [previous tutorial](first) and this one, the returned value will be zero, since all state will have been cleared.

Cool! We can now actually interact with the contract by sending a transaction to it.

## Sending a transaction

We will now [send a transaction](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#methods-mymethod-send) to `increase` the value of our Counter. Sending a transaction is not as straightforward as making a call; we need to specify who the sender will be, the gas limit, and the gas price we are going to use. To keep this example simple, we will use a hardcoded value for both gas and gas price and send the transaction from the first available account on the node.

> Note: In a real-world application, you may want to [estimate the gas](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#methods-mymethod-estimategas) of your transactions and check a [gas price oracle](https://ethgasstation.info/) to know the optimal values to use on every transaction.

Let's increase our `Counter` value by 20 every time our snippet is run and then use the code we had written before to display the updated value:

```js
// Retrieve accounts from the local node, we will use the first one to send the transaction
const accounts = await web3.eth.getAccounts();

// Send a transaction to increase() the Counter contract
await counter.methods.increase(20)
  .send({ from: accounts[0], gas: 50000, gasPrice: 1e6 });

// Call the value() function of the deployed Counter contract
const value = await counter.methods.value().call();
console.log(value);
```

We can now run the snippet and check that the counter's value is increased every time we call it.

```console
$ node src/index.js
31
$ node src/index.js
51
$ node src/index.js
71
```

You can also try interacting with the contract using `zos send-tx` and `zos call` as we did in the previous tutorial and verify that it is the same instance we are working with from two different interfaces.

The snippet from this tutorial, while simple, is the basis for interacting with your smart contracts from your javascript applications. Remember you can use other libraries other than `web3.js` - or even other languages other than javascript! ZeppelinOS will take care of managing your contracts on the blockchain. 

In the next tutorial, we will go into a more interesting, smart contract application. We will work with more complex logic, connect with `@openzeppelin/contracts` to create a token, and connect different contracts between themselves.