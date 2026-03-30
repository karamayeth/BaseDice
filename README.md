# BaseDice
BaseDice.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract BaseDice {
    address public immutable owner;
    uint256 public treasuryBalance;

    event DiceRolled(
        address indexed player,
        uint8 roll,
        uint256 bet,
        uint256 payout,
        string message,
        uint256 timestamp
    );

    string[6] private messages = [
        unicode"太可惜了，下次再来！",
        unicode"1 点... 运气一般～",
        unicode"2 点，继续努力！",
        unicode"3 点，中规中矩",
        unicode"4 点，小赢一把！",
        unicode"5 点，欧气满满！"
    ];

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this");
        _;
    }

    function roll() external payable {
        require(msg.value == 0.001 ether, "Must pay exactly 0.001 ETH to roll");

        // 伪随机生成 1~6 点
        uint256 random = uint256(keccak256(abi.encodePacked(
            block.prevrandao,
            msg.sender,
            block.timestamp,
            block.number
        ))) % 6 + 1;

        uint8 rollResult = uint8(random);
        uint256 payout = 0;

        if (rollResult == 6) {
            payout = 0.005 ether;           // 5倍
        } else if (rollResult >= 4) {
            payout = 0.002 ether;           // 2倍
        } else {
            treasuryBalance += msg.value;   // 输了进入金库
        }

        string memory message = messages[rollResult - 1];

        if (payout > 0) {
            payable(msg.sender).transfer(payout);
        }

        emit DiceRolled(msg.sender, rollResult, msg.value, payout, message, block.timestamp);
    }

    function getTreasury() external view returns (uint256) {
        return treasuryBalance;
    }

    function withdrawTreasury() external onlyOwner {
        uint256 amount = treasuryBalance;
        treasuryBalance = 0;
        payable(owner).transfer(amount);
    }

    function getContractInfo() external pure returns (string memory) {
        return "BaseDice - Pay 0.001 ETH to roll a 6-sided die and win up to 5x!";
    }
}
