• [What’s in an upgrade](#index1)  
• [Upgrading using the Upgrades Plugins](#index2)  
• [How upgrades work](#index3)  
• [Initialization](#index4)  
• [Upgrading](#index5)  
• [Testing](#index6)   
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• What’s in an upgrade</span>  
Smart contracts deployed using OpenZeppelin Upgrades Plugins can be upgraded to modify their code, while preserving their address, state, and balance. This allows you to iteratively add new features to your project, or fix any bugs you may find in production.

Smart contracts in Ethereum are immutable by default. Once you create them there is no way to alter them, effectively acting as an unbreakable contract among participants.

However, for some scenarios, it is desirable to be able to modify them. Think of a traditional contract between two parties: if they both agreed to change it, they would be able to do so. On Ethereum, they may desire to alter a smart contract to fix a bug they found (which might even lead to a hacker stealing their funds!), to add additional features, or simply to change the rules enforced by it.

# <span id='index2'>• Upgrading using the Upgrades Plugins</span>  
Whenever you deploy a new contract using deployProxy in the OpenZeppelin Upgrades Plugins, that contract instance can be upgraded later. By default, only the address that originally deployed the contract has the rights to upgrade it.

deployProxy will create the following transactions:
Deploy the implementation contract (our Box contract)
Deploy the ProxyAdmin contract (the admin for our proxy).
Deploy the proxy contract and run any initializer function.

Let’s see how it works, by deploying an upgradeable version of our Box contract, using the same setup as when we deployed earlier:
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Box {
    uint256 private _value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    // Stores a new value in the contract
    function store(uint256 value) public {
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

Install the Hardhat Upgrades plugin.
```
npm install --save-dev @openzeppelin/hardhat-upgrades
```

We then need to configure Hardhat to use our @openzeppelin/hardhat-upgrades plugin. To do this add the plugin in your hardhat.config.js file as follows.
```
// hardhat.config.js
...
require('@nomiclabs/hardhat-ethers');
require('@openzeppelin/hardhat-upgrades');
...
module.exports = {
...
};
```

In order to upgrade a contract like Box we need to first deploy it as an upgradeable contract, which is a different deployment procedure than we’ve seen so far. We will initialize our Box contract by calling store with the value 42.

Hardhat doesn’t currently have a native deployment system, instead we use scripts to deploy contracts.

We will create a script to deploy our upgradeable Box contract using deployProxy. We will save this file as scripts/deploy_upgradeable_box.js.
```
// scripts/deploy_upgradeable_box.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await upgrades.deployProxy(Box, [42], { initializer: 'store' });
  await box.deployed();
  console.log('Box deployed to:', box.address);
}

main();
```

We can then deploy our upgradeable contract.

Using the run command, we can deploy the Box contract to the development network.
```
npx hardhat run --network localhost scripts/deploy_upgradeable_box.js
Deploying Box...
Box deployed to: 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0
```

We can then interact with our Box contract to retrieve the value that we stored during initialization.

We will use the Hardhat console to interact with our upgraded Box contract.

We need to specify the address of our proxy contract from when we deployed our Box contract.
```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0');
undefined
> (await box.retrieve()).toString();
'42'
```

After creating the Solidity file, we can now upgrade the instance we had deployed earlier using the upgradeProxy function.

upgradeProxy will create the following transactions:
Deploy the implementation contract (our BoxV2 contract)
Call the ProxyAdmin to update the proxy contract to use the new implementation.

We will create a script to upgrade our Box contract to use BoxV2 using upgradeProxy. We will save this file as scripts/upgrade_box.js. We need to specify the address of our proxy contract from when we deployed our Box contract.
```
// scripts/upgrade_box.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const BoxV2 = await ethers.getContractFactory('BoxV2');
  console.log('Upgrading Box...');
  await upgrades.upgradeProxy('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0', BoxV2);
  console.log('Box upgraded');
}

main();
```

We can then deploy our upgradeable contract.

Using the run command, we can upgrade the Box contract on the development network.

```
$ npx hardhat run --network localhost scripts/upgrade_box.js
Compiling 1 file with 0.8.4
Compilation finished successfully
Upgrading Box...
Box upgraded
```

Done! Our Box instance has been upgraded to the latest version of the code, while keeping its state and the same address as before. We didn’t need to deploy a new one at a new address, nor manually copy the value from the old Box to the new one.

Let’s try it out by invoking the new increment function, and checking the value afterwards:

We need to specify the address of our proxy contract from when we deployed our Box contract.
```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const BoxV2 = await ethers.getContractFactory('BoxV2');
undefined
> const box = await BoxV2.attach('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0');
undefined
> await box.increment();
...
> (await box.retrieve()).toString();
'43'
```

That’s it! Notice how the value of the Box was preserved throughout the upgrade, as well as its address. And this process is the same regardless of whether you are working on a local blockchain, a testnet, or the main network.

Let’s see how the OpenZeppelin Upgrades Plugins accomplish this.

# <span id='index3'>• How upgrades work</span>  
This section will be more theory-heavy than others: feel free to skip over it and return later if you are curious.

When you create a new upgradeable contract instance, the OpenZeppelin Upgrades Plugins actually deploys three contracts:
The contract you have written, which is known as the implementation contract containing the logic.
A ProxyAdmin to be the admin of the proxy.
A proxy to the implementation contract, which is the contract that you actually interact with.

Here, the proxy is a simple contract that just delegates all calls to an implementation contract. A delegate call is similar to a regular call, except that all code is executed in the context of the caller, not of the callee. Because of this, a transfer in the implementation contract’s code will actually transfer the proxy’s balance, and any reads or writes to the contract storage will read or write from the proxy’s own storage.

This allows us to decouple a contract’s state and code: the proxy holds the state, while the implementation contract provides the code. And it also allows us to change the code by just having the proxy delegate to a different implementation contract.

You can have multiple proxies using the same implementation contract, so you can save gas using this pattern if you plan to deploy multiple copies of the same contract.
Any user of the smart contract always interacts with the proxy, which never changes its address. This allows you to roll out an upgrade or fix a bug without requesting your users to change anything on their end - they just keep interacting with the same address as always.

If you want to learn more about how OpenZeppelin proxies work, check out Proxies.

# <span id='index4'>• Initialization</span>  
Upgradeable contracts cannot have a constructor. To help you run initialization code, OpenZeppelin Contracts provides the Initializable base contract that allows you to tag a method as initializer, ensuring it can be run only once.

As an example, let’s write a new version of the Box contract with an initializer, storing the address of an admin who will be the only one allowed to change its contents.
```
// contracts/AdminBox.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract AdminBox is Initializable {
    uint256 private _value;
    address private _admin;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    function initialize(address admin) public initializer {
        _admin = admin;
    }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() initializer {}

    // Stores a new value in the contract
    function store(uint256 value) public {
        require(msg.sender == _admin, "AdminBox: not admin");
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

When deploying this contract, we will need to specify the initializer function name (only when the name is not the default of initialize) and provide the admin address that we want to use.
```
// scripts/deploy_upgradeable_adminbox.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const AdminBox = await ethers.getContractFactory('AdminBox');
  console.log('Deploying AdminBox...');
  const adminBox = await upgrades.deployProxy(AdminBox, ['0xACa94ef8bD5ffEE41947b4585a84BdA5a3d3DA6E'], { initializer: 'initialize' });
  await adminBox.deployed();
  console.log('AdminBox deployed to:', adminBox.address);
}

main();
```

For all practical purposes, the initializer acts as a constructor. However, keep in mind that since it’s a regular function, you will need to manually call the initializers of all base contracts (if any).

You may have noticed that we included a constructor as well as an initializer. This constructor serves the purpose of leaving the implementation contract in an initialized state, which is a mitigation against certain potential attacks.

To learn more about this and other caveats when writing upgradeable contracts, check out our Writing Upgradeable Contracts guide.

# <span id='index5'>• Upgrading</span>  
Due to technical limitations, when you upgrade a contract to a new version you cannot change the storage layout of that contract.

This means that, if you have already declared a state variable in your contract, you cannot remove it, change its type, or declare another variable before it. In our Box example, it means that we can only add new state variables after value.
```
// contracts/Box.sol
contract Box {
    uint256 private _value;

    // We can safely add a new variable after the ones we had declared
    address private _owner;

    // ...
}
```

Fortunately, this limitation only affects state variables. You can change the contract’s functions and events as you wish.

If you accidentally mess up with your contract’s storage layout, the Upgrades Plugins will warn you when you try to upgrade.

# <span id='index6'>• Testing</span>  
To test upgradeable contracts we should create unit tests for the implementation contract, along with creating higher level tests for testing interaction via the proxy. We can use deployProxy in our tests just like we do when we deploy.

When we want to upgrade, we should create unit tests for the new implementation contract, along with creating higher level tests for testing interaction via the proxy after we upgrade using upgradeProxy, checking that state is maintained across upgrades.

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
