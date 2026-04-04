# BaseDice
BaseDice.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/**
 * @title BaseDiceRoller - Fun Dice Roller (Pure Entertainment Edition)
 * @notice This is a completely original, pure entertainment on-chain dice rolling game made exclusively for you.
 *         Players can roll a 6-sided die for free anytime. No betting, no payouts, no gambling.
 *         Each roll shows a fun message based on the result. Great for casual fun and sharing rolls with friends.
 */

contract BaseDiceRoller {
    address public owner;

    // Player stats
    mapping(address => uint256) public totalRolls;
    mapping(address => uint256) public bestRoll;        // Highest number rolled by player
    mapping(address => uint256) public lastRollTime;

    // Fun messages for each dice result (1-6)
    string[6] private messages = [
        "Too bad! Better luck next time!",
        "Rolled a 1... Not the best, but keep going!",
        "Rolled a 2. Keep practicing!",
        "Rolled a 3. Pretty average!",
        "Rolled a 4. Nice! Small win vibes!",
        "Rolled a 5. You're on fire today!"
    ];

    // Events
    event DiceRolled(
        address indexed player,
        uint8 roll,
        string message,
        uint256 totalRollsByPlayer,
        uint256 timestamp
    );

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this");
        _;
    }

    /**
     * @notice Roll the dice for free (pure fun, no payment required)
     */
    function rollDice() external {
        // Optional cooldown to prevent spam (can be removed if you want unlimited rolls)
        // require(block.timestamp >= lastRollTime[msg.sender] + 30 seconds, "Please wait a bit between rolls");

        lastRollTime[msg.sender] = block.timestamp;
        totalRolls[msg.sender] += 1;

        // On-chain pseudo-random roll (1 to 6)
        uint256 random = uint256(
            keccak256(
                abi.encodePacked(
                    block.prevrandao,
                    msg.sender,
                    block.timestamp,
                    block.number,
                    totalRolls[msg.sender]
                )
            )
        ) % 6 + 1;

        uint8 rollResult = uint8(random);

        // Update best roll
        if (rollResult > bestRoll[msg.sender]) {
            bestRoll[msg.sender] = rollResult;
        }

        string memory message = messages[rollResult - 1];

        emit DiceRolled(
            msg.sender,
            rollResult,
            message,
            totalRolls[msg.sender],
            block.timestamp
        );
    }

    /**
     * @notice View your own dice statistics
     */
    function getMyStats() external view returns (
        uint256 totalRollsCount,
        uint256 highestRoll,
        uint256 lastRollTimestamp
    ) {
        return (
            totalRolls[msg.sender],
            bestRoll[msg.sender],
            lastRollTime[msg.sender]
        );
    }

    /**
     * @notice View any player's dice statistics
     */
    function getPlayerStats(address player) external view returns (
        uint256 totalRollsCount,
        uint256 highestRoll,
        uint256 lastRollTimestamp
    ) {
        return (
            totalRolls[player],
            bestRoll[player],
            lastRollTime[player]
        );
    }

    /**
     * @notice Get a fun message for a specific roll result (for frontend use)
     */
    function getMessageForRoll(uint8 roll) external pure returns (string memory) {
        require(roll >= 1 && roll <= 6, "Roll must be between 1 and 6");
        return messages[roll - 1];
    }

    /**
     * @notice Owner can reset a player's stats (for testing or cleanup)
     */
    function resetPlayerStats(address player) external onlyOwner {
        totalRolls[player] = 0;
        bestRoll[player] = 0;
        lastRollTime[player] = 0;
    }

    /**
     * @notice Withdraw any accidentally sent ETH
     */
    function withdraw() external onlyOwner {
        uint256 balance = address(this).balance;
        if (balance > 0) {
            payable(owner).transfer(balance);
        }
    }

    // Receive ETH (in case someone sends by mistake)
    receive() external payable {}
}
