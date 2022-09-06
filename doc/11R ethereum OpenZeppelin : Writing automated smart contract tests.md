• [introduce](#index1)  
• [About testing](#index2)  
• [Setting up a testing environment](#index3)  
• [Writing unit tests](#index4)  
• [Performing complex assertions](#index5)   
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
In a blockchain environment, a single mistake could cost you all of your funds - or even worse, your users' funds! This guide will help you develop robust applications by writing automated tests that verify your application behaves exactly as you intended.

# <span id='index2'>• About testing</span>  
There is a wide range of testing techniques, from simple manual verifications to complex end-to-end setups, all of them useful in their own way.

When it comes to smart contract development though, practice has shown that contract unit testing is exceptionally worthwhile. These tests are simple to write and quick to run, and let you add features and fix bugs in your code with confidence.

Smart contract unit testing consists of multiple small, focused tests, which each check a small part of your contract for correctness. They can often be expressed in single sentences that make up a specification, such as 'the admin is able to pause the contract', 'transferring tokens emits an event' or 'non-admins cannot mint new tokens'.

# <span id='index3'>• Setting up a testing environment</span>  
You may be wondering how we’re going to run these tests, since smart contracts are executed inside a blockchain. Using the actual Ethereum network would be very expensive, and while testnets are free, they are also slow (with blocktimes between 5 and 20 seconds). If we intend to run hundreds of tests whenever we make a change to our code, we need something better.

What we will use is called a local blockchain: a slimmed down version of the real thing, disconnected from the Internet, running on your machine. This will simplify things quite a bit: you won’t need to get Ether, and new blocks will be mined instantly.

# <span id='index4'>• Writing unit tests</span>  
We’ll use Chai assertions for our unit tests.
```
$ npm install --save-dev chai
```

We will keep our test files in a test directory. Tests are best structured by mirroring the contracts directory: for each .sol file there, create a corresponding test file.

Time to write our first tests! These will test properties of the Box contract from previous guides: a simple contract that lets you retrieve a value the owner previously store d.

Create a test directory in your project root. We will save the test as test/Box.test.js. Each test .js file commonly has the tests for a single contract, and is named after it.
```
// test/Box.test.js
// Load dependencies
const { expect } = require('chai');

// Start test block
describe('Box', function () {
  before(async function () {
    this.Box = await ethers.getContractFactory('Box');
  });

  beforeEach(async function () {
    this.box = await this.Box.deploy();
    await this.box.deployed();
  });

  // Test case
  it('retrieve returns a value previously stored', async function () {
    // Store a value
    await this.box.store(42);

    // Test if the returned value is the same one
    // Note that we need to use strings to compare the 256 bit integers
    expect((await this.box.retrieve()).toString()).to.equal('42');
  });
});
```

We are now ready to run our tests!

Running npx hardhat test will execute all tests in the test directory, checking that your contracts work the way you meant them to:
```
npx hardhat test


  Box
    ✓ retrieve returns a value previously stored


  1 passing (578ms)
```
It’s also a very good idea at this point to set up a Continuous Integration service such as CircleCI to make your tests run automatically every time you commit your code to GitHub.

# <span id='index5'>• Performing complex assertions</span>  
Many interesting properties of your contracts may be hard to capture, such as:

verifying that the contract reverts on errors

measuring by how much an account’s Ether balance changed

checking that the proper events are emitted

OpenZeppelin Test Helpers is a library designed to help you test all of these properties. It will also simplify the tasks of simulating time passing on the blockchain and handling very large numbers.

To install the OpenZeppelin Test Helpers, run:
```
$ npm install --save-dev @openzeppelin/test-helpers
```
We can then update our tests to use OpenZeppelin Test Helpers for very large number support, to check for an event being emitted and to check that a transaction reverts.
```
// test/Box.test.js
// Load dependencies
const { expect } = require('chai');

// Import utilities from Test Helpers
const { BN, expectEvent, expectRevert } = require('@openzeppelin/test-helpers');

// Load compiled artifacts
const Box = artifacts.require('Box');

// Start test block
contract('Box', function ([ owner, other ]) {
  // Use large integers ('big numbers')
  const value = new BN('42');

  beforeEach(async function () {
    this.box = await Box.new({ from: owner });
  });

  it('retrieve returns a value previously stored', async function () {
    await this.box.store(value, { from: owner });

    // Use large integer comparisons
    expect(await this.box.retrieve()).to.be.bignumber.equal(value);
  });

  it('store emits an event', async function () {
    const receipt = await this.box.store(value, { from: owner });

    // Test that a ValueChanged event was emitted with the new value
    expectEvent(receipt, 'ValueChanged', { value: value });
  });

  it('non owner cannot store a value', async function () {
    // Test a transaction reverts
    await expectRevert(
      this.box.store(value, { from: other }),
      'Ownable: caller is not the owner',
    );
  });
});
```

These will test properties of the Ownable Box contract from previous guides: a simple contract that lets you retrieve a value the owner previously stored.

Run your tests again to see the Test Helpers in action:
```
npx truffle test
...
  Contract: Box
    ✓ retrieve returns a value previously stored
    ✓ store emits an event
    ✓ non owner cannot store a value (588ms)


  3 passing (753ms)
```

The Test Helpers will let you write powerful assertions without having to worry about the low-level details of the underlying Ethereum libraries. To learn more about what you can do with them, head to their API reference.

# <span id='index98'>• OpenZeppelin Tutorials 教程</span>  
CN 中文 Github  [OpenZeppelin 教程 : github.com/565ee/OpenZeppelin_CN](https://github.com/565ee/OpenZeppelin_CN)  
CN 中文 CSDN    [OpenZeppelin 教程 : blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118/category_11997496.html)  
EN 英文 Github  [OpenZeppelin Tutorials : github.com/565ee/OpenZeppelin_EN](https://github.com/565ee/OpenZeppelin_EN)  

# <span id='index99'>• Contact 联系方式</span>  
Homepage   : [565.ee](https://565.ee)  
GitHub     : [github.com/565ee](https://github.com/565ee)  
Email      : 565.eee@gmail.com  
Facebook   : [facebook.com/565.ee](https://facebook.com/565.ee)  
Twitter    : [twitter.com/565_eee](https://twitter.com/565_eee)  
Telegram   : [t.me/ee_565](https://t.me/ee_565)
