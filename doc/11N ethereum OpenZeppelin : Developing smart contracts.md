• [Setting up a Project](#index1)  
• [First contract](#index2)  
• [Compiling Solidity](#index3)  
• [Adding more contracts](#index4)  
• [Using OpenZeppelin Contracts](#index5)  
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• Setting up a Project</span>  
The first step after creating a project is to install a development tool.

The most popular development framework for Ethereum is Hardhat, and we cover its most common use with ethers.js. The next most popular is Truffle which uses web3.js. Each has their strengths and it is useful to be comfortable using all of them.

In these guides we will show how to develop, test and deploy smart contracts using Truffle and Hardhat.

To get started with Hardhat we will install it in our project directory.
```
$ npm install --save-dev hardhat
```
Once installed, we can run npx hardhat. This will create a Hardhat config file (hardhat.config.js) in our project directory.

# <span id='index2'>• First contract</span>  
We store our Solidity source files (.sol) in a contracts directory. This is equivalent to the src directory you may be familiar with from other languages.

We can now write our first simple smart contract, called Box: it will let people store a value that can be later retrieved.

We will save this file as contracts/Box.sol. Each .sol file should have the code for a single contract, and be named after it.

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

# <span id='index3'>• Compiling Solidity</span>  
The Ethereum Virtual Machine (EVM) cannot execute Solidity code directly: we first need to compile it into EVM bytecode.

Our Box.sol contract uses Solidity 0.8 so we need to first configure Hardhat to use an appropriate solc version.

We specify a Solidity 0.8 solc version in our hardhat.config.js.

```
// hardhat.config.js

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
 module.exports = {
  solidity: "0.8.4",
};
```

If you’re unfamiliar with the npx command, check out our Node project setup guide.
```
npx hardhat compile
Solidity 0.8.4 is not fully supported yet. You can still use Hardhat, but some features, like stack traces, might not work correctly.

Learn more at https://hardhat.org/reference/solidity-support"

Compiling 1 file with 0.8.4
Compilation finished successfully
```

The compile built-in task will automatically look for all contracts in the contracts directory, and compile them using the Solidity compiler using the configuration in hardhat.config.js.

You will notice an artifacts directory was created: it holds the compiled artifacts (bytecode and metadata), which are .json files. It’s a good idea to add this directory to your .gitignore.

# <span id='index4'>• Adding more contracts</span>  
As your project grows, you will begin to create more contracts that interact with each other: each one should be stored in its own .sol file.

To see how this looks, let’s add a simple access control system to our Box contract: we will store an administrator address in a contract called Auth, and only let Box be used by those accounts that Auth allows.

Because the compiler will pick up all files in the contracts directory and subdirectories, you are free to organize your code as you see fit. Here, we’ll store the Auth contract in an access-control subdirectory:

```
// contracts/access-control/Auth.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Auth {
    address private _administrator;

    constructor(address deployer) {
        // Make the deployer of the contract the administrator
        _administrator = deployer;
    }

    function isAdministrator(address user) public view returns (bool) {
        return user == _administrator;
    }
}
```

Separating concerns across multiple contracts is a great way to keep each one simple, and is generally a good practice.

However, this is not the only way to split your code into modules. You can also use inheritance for encapsulation and code reuse in Solidity, as we’ll see next.

# <span id='index5'>• Using OpenZeppelin Contracts</span>  
Reusable modules and libraries are the cornerstone of great software. OpenZeppelin Contracts contains lots of useful building blocks for smart contracts to build on. And you can rest easy when building on them: they’ve been the subject of multiple audits, with their security and correctness battle-tested.

Importing OpenZeppelin Contracts
The latest published release of the OpenZeppelin Contracts library can be downloaded by running:
```
$ npm install @openzeppelin/contracts
```

To use one of the OpenZeppelin Contracts, import it by prefixing its path with @openzeppelin/contracts. For example, in order to replace our own Auth contract, we will import @openzeppelin/contracts/access/Ownable.sol to add access control to Box:
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Ownable from the OpenZeppelin Contracts library
import "@openzeppelin/contracts/access/Ownable.sol";

// Make Box inherit from the Ownable contract
contract Box is Ownable {
    uint256 private _value;

    event ValueChanged(uint256 value);

    // The onlyOwner modifier restricts who can call the store function
    function store(uint256 value) public onlyOwner {
        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

The OpenZeppelin Contracts documentation is a great place to learn about developing secure smart contract systems. It features both guides and a detailed API reference: see for example the Access Control guide to know more about the Ownable contract used in the code sample above.

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
