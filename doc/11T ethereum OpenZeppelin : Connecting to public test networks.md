• [introduce](#index1)  
• [Accessing a testnet node](#index2)  
• [Creating a new account](#index3)  
• [Configuring the network](#index4)  
• [Funding the testnet account](#index5)  
• [Working on a testnet](#index6)    
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
After you have written your contracts, and tried them out locally and tested them thoroughly, it’s time to move to a persistent public testing environment, where you and your beta users can start interacting with your application.

We will use public testing networks (aka testnets) for this, which are networks that operate similar to the main Ethereum network, but where Ether has no value and is free to acquire - making them ideal for testing your contracts at no cost.

Remember that deploying to a public test network is a necessary step when developing an Ethereum project. They provide a safe environment for testing that closely mimics the main network - you don’t want to take out your project for a test drive in a network where mistakes will cost you and your users money!

# <span id='index2'>• Accessing a testnet node</span>  
While you can spin up your own Geth or OpenEthereum node connected to a testnet, the easiest way to access a testnet is via a public node service such as Alchemy or Infura. Alchemy and Infura provide access to public nodes for all testnets and the main network, via both free and paid plans.

We say a node is public when it can be accessed by the general public, and manages no accounts. This means that it can reply to queries and relay signed transactions, but cannot sign transactions on its own.

In this guide we will use Alchemy, though you can use Infura, or another public node provider of your choice.

Head over to Alchemy (includes referral code), sign up, and jot down your assigned API key - we will use it later to connect to the network.

# <span id='index3'>• Creating a new account</span>  
To send transactions in a testnet, you will need a new Ethereum account. There are many ways to do this: here we will use the mnemonics package, which will output a fresh mnemonic (a set of 12 words) we will use to derive our accounts:
```
npx mnemonics
drama film snack motion ...
```
Make sure to keep your mnemonic secure. Do not commit secrets to version control. Even if it is just for testing purposes, there are still malicious users out there who will wreak havoc on your testnet deployment for fun!

# <span id='index4'>• Configuring the network</span>  
Since we are using public nodes, we will need to sign all our transactions locally. We will configure the network with our mnemonic and an Alchemy endpoint.

We need to update our configuration file with a new network connection to the testnet. Here we will use Rinkeby, but you can use whichever you want:
```
// hardhat.config.js
+ const { alchemyApiKey, mnemonic } = require('./secrets.json');
...
  module.exports = {
+    networks: {
+     rinkeby: {
+       url: `https://eth-rinkeby.alchemyapi.io/v2/${alchemyApiKey}`,
+       accounts: { mnemonic: mnemonic },
+     },
+   },
...
};
```

Note in the first line that we are loading the project id and mnemonic from a secrets.json file, which should look like the following, but using your own values. Make sure to .gitignore it to ensure you don’t commit secrets to version control!
```
{
  "mnemonic": "drama film snack motion ...",
  "alchemyApiKey": "JPV2..."
}
```

We can now test out that this configuration is working by listing the accounts we have available for the Rinkeby network. Remember that yours will be different, as they depend on the mnemonic you used.
```
npx hardhat console --network rinkeby
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> accounts = await ethers.provider.listAccounts()
[
  '0xEce6999C6c5BDA71d673090144b6d3bCD21d13d4',
  '0xC1310ade58A75E6d4fCb8238f9559188Ea3808f9',
...
]
```

# <span id='index5'>• Funding the testnet account</span>  
Most public testnets have a faucet: a site that will provide you with a small amount of test Ether for free. If you are on Rinkeby, head on to the Rinkeby Authenticated Faucet to get funds by authenticating with your Twitter or Facebook account. Alternatively, you can also use MetaMask’s faucet to ask for funds directly to your MetaMask accounts.

Armed with a funded account, let’s deploy our contracts to the testnet!

# <span id='index6'>• Working on a testnet</span>  
With a project configured to work on a public testnet, we can now finally deploy our Box contract. The command here is exactly the same as if you were on your local development network, though it will take a few seconds to run as new blocks are mined.

```
npx hardhat run --network rinkeby scripts/deploy.js
Deploying Box...
Box deployed to: 0xD7fBC6865542846e5d7236821B5e045288259cf0
```
That’s it! Your Box contract instance will be forever stored in the testnet, and publicly accessible to anyone.

You can see your contract on a block explorer such as Etherscan. Remember to access the explorer on the testnet where you deployed your contract, such as rinkeby.etherscan.io for Rinkeby.

You can also interact with your instance as you regularly would, either using the console, or programmatically.
```
npx hardhat console --network rinkeby
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0xD7fBC6865542846e5d7236821B5e045288259cf0');
undefined
> await box.store(42);
{
  hash: '0x330e331d30ee83f96552d82b7fdfa6156f9f97d549a612eeef7283d18b31d107',
...
> (await box.retrieve()).toString()
'42'
```

Keep in mind that every transaction will cost some gas, so you will eventually need to top up your account with more funds.

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
