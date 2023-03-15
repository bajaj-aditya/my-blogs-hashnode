---
title: "Ethernaut - Walkthrough for Noobs - 4 - Telephone"
datePublished: Wed Mar 15 2023 15:31:26 GMT+0000 (Coordinated Universal Time)
cuid: clf9ub67z000009ky9m83eaeu
slug: ethernaut-walkthrough-for-noobs-4-telephone
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678894210348/832df83a-19e5-4355-8112-5b5aeb961783.jpeg
tags: ethereum, smart-contracts, ethereum-smart-contracts, ethernaut, smart-contract-security

---

This challenge again is fairly simple and teaches us a new concept. As usual, I will break down the concepts and give you all the resources and tools to solve the problem on your own and then give a solution in case you are stuck.

> Objective: Claim ownership of the contract to complete this level.

### Background

If you have used solidity in any capacity whatsoever, you must have come across something called `msg.sender` and `tx.origin`. They are special variables of type address in the global namespace. `msg.sender` gives the address of the sender of the current call chain whereas `tx.origin` gives the sender of the transaction.

For Ex: In a simple call chain A-&gt;B-&gt;C-&gt;D, Inside D `msg.sender` will be C, and `tx.origin` will be A.

*Note:* `tx.origin` *must be an EOA and can never be another contract.*

### Problems and Code of Importance

The code and solution are extremely small and easy to understand.

The only function that can update the `owner` is the `changeOwner()` function, so naturally that should be the focus of the attack.

```solidity
function changeOwner(address _owner) public {
    //only changes the owner if the call is not from original EOA.
   if (tx.origin != msg.sender) {
     owner = _owner;
   }
 }
```

Now, how to exploit this.

Let's consider the following case:

Case 1: Aditya(EOA) calls `Telephone.changeOwner(Bajaj)`

* `tx.origin` == Aditya(EOA)
    
* `msg.sender` == Aditya(EOA)
    

The owner will not change.

Case 2: Aditya(EOA) calls `ContractExploit.something()` which further calls `Telephone.changeOwner(Bajaj)`

Here, Inside `Telephone.changeOwner`

* `tx.origin` == Aditya(EOA)
    
* `msg.sender` == `ContractExploit.something()`
    

Hence, the owner will be changed to our desired address.

Thus, we need to only create a middle contract between the EOA and the Telephone contract.

### Solution

* Open up Remix and create two files, Telephone.sol and Exploit.sol
    
* Copy the ethernaut code in Telephone.sol
    
* Add the below code to your Exploit.sol (Explanation in comments)
    
* ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;
    import "./Telephone.sol";
    
    contract Exploit {
        Telephone telephone = Telephone("level instance without quotes"); //intializes a variable telephone and stores the ethernaut instance.
        function exploit() public {
            telephone.changeOwner(msg.sender);
        } //exploit calls the changeOwner function in the telephone contract. 
    }
    ```
    
* To understand what's happening here, essentially from the perspective of `Exploit.exploit()`, We are the `msg.sender` and `tx.origin` but from the perspective of `telephone.changeOwner()` we are `tx.origin` but `Exploit.exploit()` is `msg.sender` hence, fulfilling our condition.
    
* Compile and Deploy both contracts in the Injected Web3 Metamask Env.
    
* Call the exploit() function.
    
* Check on the console whether you are the owner of the instance.
    
* Submit Instance.
    

Here are additional resources, to understand `tx.origin` attacks. Also as a rule you should never use `tx.origin` in your contract unless absolutely required. It might get deprecated in a future update, and opens up the contract to potential phishing attacks.

1. [Point 16](https://blog.sigmaprime.io/solidity-security.html#tx-origin)
    
2. [Detailed Explanation](https://blog.finxter.com/tx-origin-phishing-attack-smart-contract-security-series-part-4/#:~:text=Similarly%2C%20the%20case%20of%20smart,should%20be%20able%20to%20call.)
    
3. [For those who watch videos](https://youtu.be/mk4wDlVB4ro)
    
    Telephone✅