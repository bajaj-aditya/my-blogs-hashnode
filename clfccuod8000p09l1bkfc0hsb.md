---
title: "Ethernaut - Walkthrough for Noobs - 5 - Token"
datePublished: Fri Mar 17 2023 09:46:01 GMT+0000 (Coordinated Universal Time)
cuid: clfccuod8000p09l1bkfc0hsb
slug: ethernaut-walkthrough-for-noobs-5-token
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679046262527/1fb5f863-90ac-40a4-ad6b-e75549deb1b0.jpeg
tags: ethereum, smart-contracts, 2articles1week, ethernaut

---

The token challenge teaches us about overflows and underflows and if you have taken the CS50 course or equivalent you know what I am talking about. As usual, I will give you all the information to solve the challenge on your own and some good practices, and then give you a solution just in case.

> The goal of this level is for you to hack the basic token contract.
> 
> You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

### Background

In the coin flip challenge, we talked about `uint256`:

1. u stands for unsigned: meaning it can only take non-negative integers.
    
2. int: stands for Integer.
    
3. 256: bits in size.
    

The maximum value of `uint256` is equal to (2^256 -1) which is a gigantic number, to be precise it's:

> uintMAX = 115792089237316195423570985008687907853269984665640564039457584007913129639935

Okay but let's say you take a uint256 variable and assign it a higher value.

Solidity handles it not by giving you an error, it handles it by reading a very popular book by Mark Mason and starting the count again from 0. This phenomenon is called **overflow.**

Example: uintMax + 3 ==&gt; 2

The reverse of this phenomenon is called **underflow.** If you take a `uint256` and assign it a negative value, solidity starts the count from the maximum number backward.

Example: - 3 ==&gt; uintMax - 2

### Code of Importance and Problem

The token is an extremely simplified version of the ERC20 token. Explanation in comments

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {
//state variables
  mapping(address => uint) balances; // to map user balances
  uint public totalSupply; //will track the total supply

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply; //mints token and adds them to the creator wallet.
  }

//code of importance
  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0); //checks if the sender has enough funds to transfer
    balances[msg.sender] -= _value; //subtracts the sent amount 
    balances[_to] += _value; //updates the receiver balance 
    return true; 
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner]; //checks the balance of owner
  }
}
```

Now, keeping the underflow concept in mind. We have 20 tokens, given to us. If we try to invoke the transfer function by sending 21 tokens, it would result in an underflow, and the contract proceeds, while giving us plenty of tokens.

*Note: If you are using newer versions of Solidity or SafeMath, you would not face this issue and the operation would likely lead to a revert.*

### Solution

* Create a new level instance and open up the console.
    
* ```solidity
    await contract.transfer('any payable address with quotes', 21)
    ```
    
* ```solidity
    await contract.balanceOf(player) //tells us our balance
    ```
    
* Cross-check that you have a shit ton of tokens and Submit Instance.
    

Token✅

**#2Articles1Week**