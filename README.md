# DEX

**STEP ONE**
npm install -g truffle
npm install -g ganache-cli

**STEP TWO**
mkdir decentralized-exchange
cd decentralized-exchange
truffle init

**STEP THREE**
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract TokenA is ERC20 {
    constructor(uint256 initialSupply) ERC20("TokenA", "TKA") {
        _mint(msg.sender, initialSupply);
    }
}

contract TokenB is ERC20 {
    constructor(uint256 initialSupply) ERC20("TokenB", "TKB") {
        _mint(msg.sender, initialSupply);
    }
}

**STEP FOUR**
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract DEX {
    IERC20 public tokenA;
    IERC20 public tokenB;
    uint256 public liquidity;
    mapping(address => uint256) public liquidityProvided;

    event LiquidityProvided(address indexed provider, uint256 amount);
    event LiquidityRemoved(address indexed provider, uint256 amount);
    event Swap(address indexed swapper, address fromToken, address toToken, uint256 amount);

    constructor(address _tokenA, address _tokenB) {
        tokenA = IERC20(_tokenA);
        tokenB = IERC20(_tokenB);
    }

    function provideLiquidity(uint256 amountA, uint256 amountB) public {
        tokenA.transferFrom(msg.sender, address(this), amountA);
        tokenB.transferFrom(msg.sender, address(this), amountB);
        liquidity += amountA + amountB;
        liquidityProvided[msg.sender] += amountA + amountB;
        emit LiquidityProvided(msg.sender, amountA + amountB);
    }

    function removeLiquidity(uint256 amount) public {
        require(liquidityProvided[msg.sender] >= amount, "Not enough liquidity provided");
        uint256 amountA = (amount / 2);
        uint256 amountB = (amount / 2);
        tokenA.transfer(msg.sender, amountA);
        tokenB.transfer(msg.sender, amountB);
        liquidity -= amount;
        liquidityProvided[msg.sender] -= amount;
        emit LiquidityRemoved(msg.sender, amount);
    }

    function swap(address fromToken, uint256 amount) public {
        require(fromToken == address(tokenA) || fromToken == address(tokenB), "Invalid token address");
        IERC20 fromERC20 = IERC20(fromToken);
        IERC20 toERC20 = fromToken == address(tokenA) ? tokenB : tokenA;
        fromERC20.transferFrom(msg.sender, address(this), amount);
        uint256 swapAmount = getSwapAmount(fromToken, amount);
        toERC20.transfer(msg.sender, swapAmount);
        emit Swap(msg.sender, fromToken, address(toERC20), swapAmount);
    }

    function getSwapAmount(address fromToken, uint256 amount) public view returns (uint256) {
        uint256 totalSupply = tokenA.balanceOf(address(this)) + tokenB.balanceOf(address(this));
        return (amount * totalSupply) / liquidity;
    }
}

**STEP FIVE**
const TokenA = artifacts.require("TokenA");
const TokenB = artifacts.require("TokenB");
const DEX = artifacts.require("DEX");

module.exports = async function (deployer) {
    await deployer.deploy(TokenA, 1000000);
    const tokenA = await TokenA.deployed();
    await deployer.deploy(TokenB, 1000000);
    const tokenB = await TokenB.deployed();
    await deployer.deploy(DEX, tokenA.address, tokenB.address);
};

**STEP SIX**
ganache-cli

**STEP SEVEN**
truffle migrate

**STEP EIGGT**
let tokenA = await TokenA.deployed();
let tokenB = await TokenB.deployed();
let dex = await DEX.deployed();

let accounts = await web3.eth.getAccounts();
let user = accounts[0];

// Approve DEX to spend user's tokens
await tokenA.approve(dex.address, 1000, { from: user });
await tokenB.approve(dex.address, 1000, { from: user });

// Provide liquidity
await dex.provideLiquidity(500, 500, { from: user });

// Swap tokens
await dex.swap(tokenA.address, 100, { from: user });
