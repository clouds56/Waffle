[![Build Status](https://travis-ci.com/EthWorks/Waffle.svg?token=xjj4U84eSFwEsYLTc5Qe&branch=master)](https://travis-ci.com/EthWorks/Waffle)

# Ethereum Waffle
Library for writing and testing smart contracts.

Sweeter and simpler than [truffle](https://github.com/trufflesuite/truffle).

Works with [ethers-js](https://github.com/ethers-io/ethers.js/). Taste best with ES6.

## Philosophy
* __Simpler__: minimalistic, a couple of helpers, matchers and tasks rather than a framework, few dependencies.
* __Sweeter__: Nice syntax, fast, easy to extend.

## Install:
To start using with npm, type:
```sh
npm i ethereum-waffle
```

or with Yarn:
```sh
yarn add ethereum-waffle
```

## Step by step guide

### Example contract
Below is example contract written in Solidity. Place it in `contracts` directory of your project:

```solidity
pragma solidity ^0.4.24;

import "../BasicToken.sol";

contract BasicTokenMock is BasicToken {

  constructor(address initialAccount, uint256 initialBalance) public {
    balances[initialAccount] = initialBalance;
    totalSupply_ = initialBalance;
  }

}
```

### Example test
Belows is example test written for the contract above written with Waffle. Place it in `test` directory of your project:

```js
import chai from 'chai';
import {createMockProvider, deployContract, getWallets, solidity} from '../../lib/waffle';
import BasicTokenMock from './build/BasicTokenMock';

chai.use(solidity);

const {expect} = chai;

describe('Example', () => {
  let provider;
  let token;
  let wallet;
  let walletTo;

  beforeEach(async () => {
    provider = createMockProvider();
    [wallet, walletTo] = await getWallets(provider);
    token = await deployContract(wallet, BasicTokenMock, [wallet.address, 1000]);
  });

  it('Assigns initial balance', async () => {
    expect(await token.balanceOf(wallet.address)).to.eq(1000);
  });

  it('Transfer adds amount to destination account', async () => {
    await token.transfer(walletTo.address, 7);
    expect(await token.balanceOf(wallet.address)).to.eq(993);
    expect(await token.balanceOf(walletTo.address)).to.eq(7);
  });

  it('Transfer emits event', async () => {
    await expect(token.transfer(walletTo.address, 7))
      .to.emit(token, 'Transfer')
      .withArgs(wallet.address, walletTo.address, 7);
  });

  it('Can not transfer above the amount', async () => {
    await expect(token.transfer(walletTo.address, 1007)).to.be.reverted;
  });
});
```

### Compile
To compile contracts simply type:
```sh
npx waffle
```

### Run tests
To run test type in the console:
```sh
mocha
```

### Adding a task
For convince, you can add a task to your `package.json`. In the sections `scripts`, add following line:
```json
  "test": "waffle && test"
```

Now you can build and test your contracts with one command:
```sh
npm test
```

## Features walkthrough

### Import contracts from npms
Import solidity files from solidity files form npm modules that you added to your project, e.g:
```
import "openzeppelin-solidity/contracts/math/SafeMath.sol";
```

### Create a mock provider
Create a mock provider (no need to run any tests!) and test your contracts against it, e.g.:
```js
provider = createMockProvider();
```

### Get example wallets
Get wallets you can use to sign transactions:
```js
[wallet, walletTo] = await getWallets(provider);
```

### Deploy contract
Deploy a contract:
```js
token = await deployContract(wallet, BasicTokenMock, [wallet.address, 1000]);
```

### Chai matchers
A set of sweet chai matchers, makes your test easy to write and read. Below couple of examples.

* Testing big numbers:
```js
expect(await token.balanceOf(wallet.address)).to.eq(993);
```
Available matchers for BigNumbers are: `equal`, `eq`, `above`, `below`, `least`, `most`.

* Testing what events where emitted with what arguments:
```js
await expect(token.transfer(walletTo.address, 7))
  .to.emit(token, 'Transfer')
  .withArgs(wallet.address, walletTo.address, 7);
```

* Testing if transaction was reverted:
```js
await expect(token.transfer(walletTo.address, 1007)).to.be.reverted;
```

## Configuring