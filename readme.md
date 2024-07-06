✍ Submit Your Homework 3
In this homework, you'll extend the capabilities of our PayPool smart contract. You'll practice using structs to organize data, enums to represent states, and leverage block timestamps for time-dependent logic.

Key Concepts

Structs: Custom data types to group related variables.

Enums: Sets of named constants to improve code readability.

Block Timestamps: Access the current block's timestamp (block.timestamp) for time-based actions.

Instructions

1.Struct for Deposit Records:

Create a struct named DepositRecord:

depositor (address)

amount (uint256)

timestamp (uint256)

2.Enum for Deposit Statuses:

Create an enum named DepositStatus:

Pending

Approved

Rejected

3. Modify the PayPool Contract:

3.1.Store Deposit Records:

Inside the deposit function:

Get the current timestamp (block.timestamp).

Create a DepositRecord with the depositor's address, deposited amount, and timestamp.

Add this record to an array called depositHistory.

3.2.Manage Statuses:

Add a status property (type DepositStatus) to the DepositRecord struct.

Initialize new deposits as Pending.

Create owner-only functions:

approveDeposit(uint256 index) to set a deposit's status to Approved.

rejectDeposit(uint256 index) to set a deposit's status to Rejected.

3.3.Retrieve Deposit Data:

Create a function: getDepositHistory() public view returns (DepositRecord[] memory) to return the depositHistory array.

Paypool Contract

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.25;

// Imports

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract PayPool is ReentrancyGuard {

    // Data

    uint public totalBalance;

    address public owner;

    address\[\] public depositAddresses;

    mapping(address =&gt; uint256) public allowances;

    // Events

    event Deposit(address indexed depositer, uint256 amount);

    event AddressAdded(address indexed depositer);

    event AddressRemoved(address indexed depositer);

    event AllowanceGranted(address indexed user, uint amount);

    event AllowanceRemoved(address indexed user);

    event FundsRetrieved(address indexed recepient, uint amount);

    modifier isOwner() {

        require(msg.sender == owner, "Not owner!");

        \_;

    }

    modifier gotAllowance(address user) {

        require(hasAllowance(user), "This address has no allowance");

        \_;

    }

    modifier canDepositTokens(address depositer) {

        require(canDeposit(depositer), "This address is not allowed to deposit tokens");

        \_;

    }

    constructor() payable {

        totalBalance = msg.value;

        owner = msg.sender;

    }

    // Internal functions

    function hasAllowance(address user) internal view returns(bool) {

        return allowances\[user\] &gt; 0;

    }

    function canDeposit(address depositer) internal view returns(bool) {

        for (uint i = 0; i &lt; depositAddresses.length; i++) {

            if (depositAddresses\[i\] == depositer) {

                return true;

            }

        }

        return false;

    }

    // Execute Functions

    function addDepositAddress(address depositer) external isOwner {

        depositAddresses.push(depositer);

        emit AddressAdded(depositer);

    }

    function removeDepositAddress(uint index) external isOwner canDepositTokens(depositAddresses\[index\]) {

        depositAddresses\[index\] = address(0);

        emit AddressRemoved(depositAddresses\[index\]);

    }

    function deposit() external canDepositTokens(msg.sender) payable {

        totalBalance += msg.value;

        emit Deposit(msg.sender, msg.value);

    }

    function retrieveBalance() external isOwner nonReentrant {

        uint balance = totalBalance;

        (bool success, ) = [owner.call](http://owner.call){value: balance}("");

        require(success, "Transfer failed");

        totalBalance = 0;

        emit FundsRetrieved(owner, balance);

    }

    function giveAllowance(uint amount, address user) external isOwner {

        require(totalBalance &gt;= amount, "There are no enough tokens inside the pool to give allowance");

        allowances\[user\] = amount;

        unchecked {

            totalBalance -= amount;

        }

        emit AllowanceGranted(user, amount);

    }

    function removeAllowance(address user) external isOwner gotAllowance(user) {

        allowances\[user\] = 0;

        emit AllowanceRemoved(user);

    }

    function allowRetrieval() external gotAllowance(msg.sender) nonReentrant {

        uint amount = allowances\[msg.sender\];

        (bool success, ) = [msg.sender.call](http://msg.sender.call){value: amount}("");

        require(success, "Retrieval failed");

        allowances\[msg.sender\] = 0;

        emit FundsRetrieved(msg.sender, amount);

    }

}
Homework Checklist

[ ] I created a DepositRecord struct.

[ ] I added a DepositStatus enum.

[ ] I modified the deposit function to store deposit records with timestamps.

[ ] I created functions to let the owner approve or reject deposits.

[ ] I created a getDepositHistory function to retrieve deposit data.

[ ] I rigorously tested my changes in Remix IDE.

After completing the contract, deploy it on Scroll Sepolia testnet and share your github repo below.

Congratulations, now you are a Solidity Developer!

To submit your project, first connect to GitHub and then submit it from your repository.
