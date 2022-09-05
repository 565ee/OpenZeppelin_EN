• [Setting up a Local Blockchain](#index1)  
• [Deploying a Smart Contract](#index2)  
• [Interacting from the Console](#index3)  
• [Interacting programmatically](#index4)  
• [Getting a contract instance](#index5)  
• [Calling the contract](#index6)  
• [Sending a transaction](#index7)   
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• Setting up a Local Blockchain</span>  
Before we begin, we first need an environment where we can deploy our contracts. The Ethereum blockchain (often called "mainnet", for "main network") requires spending real money to use it, in the form of Ether (its native currency). This makes it a poor choice when trying out new ideas or tools.

To solve this, a number of "testnets" (for "test networks") exist: these include the Ropsten, Rinkeby, Kovan and Goerli blockchains. They work very similarly to mainnet, with one difference: you can get Ether for these networks for free, so that using them doesn’t cost you a single cent. However, you will still need to deal with private key management, blocktimes in the range of 5 to 20 seconds, and actually getting this free Ether.

During development, it is a better idea to instead use a local blockchain. It runs on your machine, requires no Internet access, provides you with all the Ether that you need, and mines blocks instantly. These reasons also make local blockchains a great fit for automated tests.

Hardhat comes with a local blockchain built-in, the Hardhat Network.

Upon startup, Hardhat Network will create a set of unlocked accounts and give them Ether.
```
$ npx hardhat node
```
Hardhat Network will print out its address, http://127.0.0.1:8545, along with a list of available accounts and their private keys.

Keep in mind that every time you run Hardhat Network, it will create a brand new local blockchain - the state of previous runs is not preserved. This is fine for short-lived experiments, but it means that you will need to have a window open running Hardhat Network for the duration of these guides.

# <span id='index2'>• Deploying a Smart Contract</span>  
In the Developing Smart Contracts guide we set up our development environment.

If you don’t already have this setup, please create and setup the project and then create and compile our Box smart contract.

With our project setup complete we’re now ready to deploy a contract. We’ll be deploying Box, from the Developing Smart Contracts guide. Make sure you have a copy of Box in contracts/Box.sol.

Hardhat doesn’t currently have a native deployment system, instead we use scripts to deploy contracts.

We will create a script to deploy our Box contract. We will save this file as scripts/deploy.js.

```
// scripts/deploy.js
async function main () {
  // We get the contract to deploy
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await Box.deploy();
  await box.deployed();
  console.log('Box deployed to:', box.address);
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

We use ethers in our script, so we need to install it and the @nomiclabs/hardhat-ethers plugin.
```
$ npm install --save-dev @nomiclabs/hardhat-ethers ethers
```
We need to add in our configuration that we are using the @nomiclabs/hardhat-ethers plugin.
```
// hardhat.config.js
require('@nomiclabs/hardhat-ethers');

...
module.exports = {
...
};
```

All done! On a real network this process would’ve taken a couple of seconds, but it is near instant on local blockchains.

# <span id='index3'>• Interacting from the Console</span>  
With our Box contract deployed, we can start using it right away.

We will use the Hardhat console to interact with our deployed Box contract on our localhost network.
```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x5FbDB2315678afecb367f032d93F642f64180aa3')
undefined
```

Sending transactions
Box's first function, store, receives an integer value and stores it in the contract storage. Because this function modifies the blockchain state, we need to send a transaction to the contract to execute it.

We will send a transaction to call the store function with a numeric value:
```
> await box.store(42)
{
  hash: '0x3d86c5c2c8a9f31bedb5859efa22d2d39a5ea049255628727207bc2856cce0d3',
```

Querying state
Box's other function is called retrieve, and it returns the integer value stored in the contract. This is a query of blockchain state, so we don’t need to send a transaction:
```
> await box.retrieve()
BigNumber { _hex: '0x2a', _isBigNumber: true }
```

# <span id='index4'>• Interacting programmatically</span>  
The console is useful for prototyping and running one-off queries or transactions. However, eventually you will want to interact with your contracts from your own code.

In this section, we’ll see how to interact with our contracts from JavaScript, and use Hardhat to run our script with our Hardhat configuration.

Setup
Let’s start coding in a new scripts/index.js file, where we’ll be writing our JavaScript code, beginning with some boilerplate, including for writing async code.
```
// scripts/index.js
async function main () {
  // Our code will go here
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

We can test our setup by asking the local node something, such as the list of enabled accounts:

```
// Retrieve accounts from the local node
const accounts = await ethers.provider.listAccounts();
console.log(accounts);
```

Run the code above using hardhat run, and check that you are getting a list of available accounts in response.
```
$ npx hardhat run --network localhost ./scripts/index.js
[
  '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  '0x70997970C51812dc3A010C7d01b50e0d17dc79C8',
...
]
```

These accounts should match the ones displayed when you started the local blockchain earlier. Now that we have our first code snippet for getting data out of a blockchain, let’s start working with our contract. Remember we are adding our code inside the main function we defined above.

# <span id='index5'>• Getting a contract instance</span>  
In order to interact with the Box contract we deployed, we’ll use an ethers contract instance.

An ethers contract instance is a JavaScript object that represents our contract on the blockchain, which we can use to interact with our contract. To attach it to our deployed contract we need to provide the contract address.
```
// Set up an ethers contract, representing our deployed Box instance
const address = '0x5FbDB2315678afecb367f032d93F642f64180aa3';
const Box = await ethers.getContractFactory('Box');
const box = await Box.attach(address);
```

# <span id='index6'>• Calling the contract</span>  
Let’s start by displaying the current value of the Box contract.

We’ll need to call the read only retrieve() public method of the contract, and await the response:
```
// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

This snippet is equivalent to the query we ran earlier from the console. Now, make sure everything is running smoothly by running the script again and checking the printed value:
```
npx hardhat run --network localhost ./scripts/index.js
Box value is 42
```

# <span id='index7'>• Sending a transaction</span>  
We’ll now send a transaction to store a new value in our Box.

Let’s store a value of 23 in our Box, and then use the code we had written before to display the updated value:
```
// Send a transaction to store() a new value in the Box
await box.store(23);

// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

We can now run the snippet, and check that the box’s value is updated!

```
$ npx hardhat run --network localhost ./scripts/index.js
Box value is 23
```

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
