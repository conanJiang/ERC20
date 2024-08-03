
Hardhat 是以太坊最流行的开发环境，它可以帮你编译和部署智能合约，并且提供了 Hardhat Network 支持本地测试和运行 Solidity。这一讲，我们将介绍如何安装 Hardhat，使用 Hardhat 编写并编译合约，并运行简单的测试。

## **Hardhat 安装**

### **安装 node**

可以使用 nvm 安装 node

<u>GitHub - nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions</u>

### **安装 Hardhat**

打开命令行工具，输入：

```javascript
mkdir hardhat-demo
cd hardhat-demo
npm init -y
npm install --save-dev hardhat
```

### **创建 Hardhat 项目**

打开命令行工具，输入：

```javascript
cd hardhat-demo
npx hardhat
```

选择第三项：创建空白项目配置 `Create an empty hardhat.config.js`

```javascript
Welcome to Hardhat v2.22.2

? What do you want to do? ...
> Create a JavaScript project
  Create a TypeScript project
  Create a TypeScript project (with Viem)
  Create an empty hardhat.config.js
  Quit
```

### **安装插件**

```javascript
npm install --save-dev @nomicfoundation/hardhat-toolbox
```

将插件添加到你的 hardhat 配置文件中 `hardhat.config.js`

```javascript
require("@nomicfoundation/hardhat-toolbox");

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.21",
};
```

如果你用过 remix，那么你直接在 remix 上点击保存的时候，会自动帮你编译的。但是在本地的 hardhat 开发环境中，你需要手动编译合约。

### **新建合约目录**

新建 `contracts` 合约目录，并添加 ERC20 合约。

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

/**
 * @dev ERC20 接口合约.
 */
interface IERC20 {
    /**
     * @dev 释放条件：当 `value` 单位的货币从账户 (`from`) 转账到另一账户 (`to`)时.
     */
    event Transfer(address indexed from, address indexed to, uint256 value);

    /**
     * @dev 释放条件：当 `value` 单位的货币从账户 (`owner`) 授权给另一账户 (`spender`)时.
     */
    event Approval(address indexed owner, address indexed spender, uint256 value);

    /**
     * @dev 返回代币总供给.
     */
    function totalSupply() external view returns (uint256);

    /**
     * @dev 返回账户`account`所持有的代币数.
     */
    function balanceOf(address account) external view returns (uint256);

    /**
     * @dev 转账 `amount` 单位代币，从调用者账户到另一账户 `to`.
     *
     * 如果成功，返回 `true`.
     *
     * 释放 {Transfer} 事件.
     */
    function transfer(address to, uint256 amount) external returns (bool);

    /**
     * @dev 返回`owner`账户授权给`spender`账户的额度，默认为0。
     *
     * 当{approve} 或 {transferFrom} 被调用时，`allowance`会改变.
     */
    function allowance(address owner, address spender) external view returns (uint256);

    /**
     * @dev 调用者账户给`spender`账户授权 `amount`数量代币。
     *
     * 如果成功，返回 `true`.
     *
     * 释放 {Approval} 事件.
     */
    function approve(address spender, uint256 amount) external returns (bool);

    /**
     * @dev 通过授权机制，从`from`账户向`to`账户转账`amount`数量代币。转账的部分会从调用者的`allowance`中扣除。
     *
     * 如果成功，返回 `true`.
     *
     * 释放 {Transfer} 事件.
     */
    function transferFrom(
        address from,
        address to,
        uint256 amount
    ) external returns (bool);
}
```

```javascript
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.10;

import "./IERC20.sol";

contract ERC20 is IERC20 {

    mapping(address => uint256) public override balanceOf;

    mapping(address => mapping(address => uint256)) public override allowance;

    uint256 public override totalSupply;   // 代币总供给

    string public name;   // 名称
    string public symbol;  // 符号
    
    uint8 public decimals = 18; // 小数位数

    // @dev 在合约部署的时候实现合约名称和符号
    constructor(string memory name_, string memory symbol_){
        name = name_;
        symbol = symbol_;
    }

    // @dev 实现`transfer`函数，代币转账逻辑
    function transfer(address recipient, uint amount) external override returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    // @dev 实现 `approve` 函数, 代币授权逻辑
    function approve(address spender, uint amount) external override returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    // @dev 实现`transferFrom`函数，代币授权转账逻辑
    function transferFrom(
        address sender,
        address recipient,
        uint amount
    ) external override returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    // @dev 铸造代币，从 `0` 地址转账给 调用者地址
    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    // @dev 销毁代币，从 调用者地址 转账给  `0` 地址
    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }

}
```

### **编译合约**

```javascript
npx hardhat compile
```

看到如下输出，说明合约编译成功：

```javascript
Compiling 2 Solidity files successfully
```

成功后，你会在文件夹下看到 `artifacts` 目录，里面的 `json` 文件就是编译结果。

## **编写单元测试**

这里的单元测试非常简单，仅包含部署合约并测试合约地址是否合法（是否部署成功）。

新建测试文件夹 `test`，在其中新建 `test.js`。单元测试中，我们会用到 `chai` 和 `ethers.js` 两个库，分别用于测试和链上交互。

```javascript
const { expect } = require('chai');
const { ethers } = require('hardhat');


describe("ERC20 合约测试", ()=>{
  it("合约部署", async () => {
     // ethers.getSigners,代表eth账号  ethers 是一个全局函数，可以直接调用
     const [owner, addr1, addr2] = await ethers.getSigners();
     // ethers.js 中的 ContractFactory 是用于部署新智能合约的抽象，因此这里的 ERC20 是我们代币合约实例的工厂。ERC20代表contracts 文件夹中的 ERC20.sol 文件
     const Token = await ethers.getContractFactory("ERC20");
     // 部署合约, 传入参数 ERC20.sol 中的构造函数参数分别是 name, symbol 这里我们都叫做WTF
     const hardhatToken = await Token.deploy("RCC", "RCC"); 
     await hardhatToken.waitForDeployment();
      // 获取合约地址
     const ContractAddress = await hardhatToken.target;
     expect(ContractAddress).to.properAddress;
  });
})
```

## **运行测试**

在命令行输入以下内容运行测试：

```javascript
npx hardhat test
# 如果有多个文件想跑指定文件可以使用
npx mocha test/test.js
```

看到如下输出，说明测试成功。

```javascript
ERC20 合约测试
    ✔ 合约部署 (1648ms)


  1 passing (2s)
```

## **部署合约**

在 remix 中，我们只需要点击一下 `deploy` 就可以部署合约了，但是在本地 hardhat 中，我们需要编写一个部署脚本。

新建一个 `scripts` 文件夹，我们来编写部署合约脚本。并在该目录下新建一个 `deploy.js`

输入以下代码

```javascript
// 我们可以通过 npx hardhat run <script> 来运行想要的脚本
// 这里你可以使用 npx hardhat run deploy.js 来运行
const hre = require("hardhat");

async function main() {
  const Contract = await hre.ethers.getContractFactory("ERC20");
  const token = await Contract.deploy("RCC","RCC");

  await token.waitForDeployment();

  console.log("成功部署合约:", token.target);
}

// 运行脚本
main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

运行以下代码部署合约到本地测试网络

hardhat 会提供一个默认的网络，参考：<u>hardhat 默认网络</u>

```javascript
npx hardhat run --network hardhat  scripts/deploy.js
```

看到如下输出，说明合约部署成功：

```javascript
(node:45779) ExperimentalWarning: stream/web is an experimental feature. This feature could change at any time
(Use `node --trace-warnings ...` to show where the warning was created)
成功部署合约: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

## **部署合约到 sepolia 测试网络 ｜ 网络配置**

### **前期准备**

1. 申请 alchemy 的 api key
2. 申请 sepolia 测试代币 <u>点击申请</u>
3. 导出私钥 因为需要把合约部署到 **sepolia** 测试网络，所以该测试账号中留有一定的测试代币。导出已有测试代币的账户的私钥，用于部署合约
4. 申请 etherscan 的 api key，用于验证合约 <u>点击申请</u>

### **配置网络**

在 `hardhat.config.js` 中，我们可以配置多个网络，这里我们配置 **sepolia** 测试网络。

编辑 `hardhat.config.js`

```javascript
require("@nomicfoundation/hardhat-toolbox");
/** @type import('hardhat/config').HardhatUserConfig */

const ALCHEMY_SEPOLIA_KEY = "";
const ETHERSCAN_API_KEY = "";
const SEPOLIA_PRIVATE_KEY = "";
module.exports = {
  solidity: "0.8.24",
  networks: {
    sepolia: {
      url: `https://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_SEPOLIA_KEY}`,
      accounts: [SEPOLIA_PRIVATE_KEY]
    },
  },
  etherscan: {
    apiKey: ETHERSCAN_API_KEY,
  },
  sourcify: {
    enabled: true
  }
};
```

配置完成运行

```javascript
npx hardhat run --network sepolia scripts/deploy.js
```

你就可以把你的合约部署到 sepolia 测试网络了。

可以通过<u>etherscan</u>查看合约部署情况

同理你也可以配置多个网络，比如 `mainnet`，`rinkeby` 等。

## **总结**

这一讲，我们介绍了 Hardhat 基础用法。通过 Hardhat 我们能够工程化 solidity 的项目，并提供了很多有用的脚手架。
