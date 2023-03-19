---
title: "Ethernaut - Walkthrough for Noobs - 7 - Force"
datePublished: Sun Mar 19 2023 10:00:44 GMT+0000 (Coordinated Universal Time)
cuid: clff89aro000309mf0jadat8o
slug: ethernaut-walkthrough-for-noobs-7-force
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679068959566/3ca049af-324b-4d36-8e8c-50289a40c999.jpeg
tags: ethereum, smart-contracts, 2articles1week, ethernaut, smart-contract-security

---

This challenge will teach you, how to shove tokens down a contract's throat, even if the contract isn't necessarily accepting them.

> The goal of this level is to make the balance of the contract greater than zero.

### Background

Usually, a contract receives ether, in a **payable** function or the [**receive**](https://docs.soliditylang.org/en/latest/contracts.html#receive-ether-function) function, or the good ol' [**fallback**](https://docs.soliditylang.org/en/latest/contracts.html#fallback-function) function. But what if a contract has none of it?

There are three more unheard-of ways to do this:

1. `selfdestruct()`: We will talk about this more, in the challenge. This little thing has created a lot of security problem.
    
2. A contract can also receive Ether as a recipient of a miner block reward.
    
3. We can also deposit funds into an address before it is deployed, since we can pre-calculate the contract address before it is generated.
    

In this challenge, we would use `selfdestruct()` , this function removes a contract from blockchain. *Note: This isn't equivalent to deleting stuff from your hard drive, your past transactions etc., will remain on the blockchain.*

It looks like this:

```solidity
selfdestruct(_address);
```

Whenever this function is called, the ether in the contract will be forcefully sent to the `_address` mentioned in the argument.

### Code of Importance - or the Lack of it.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```

This is an empty contract, without any functions, fallbacks, or constructors. I mean the cat is cute though, but doesn't really help.

The plan of action would be to create a contract, get some goerli, or sepolia in it, and self-destruct it to this address.

### Solution

* Open [Remix](https://remix.ethereum.org/).
    
* ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;
    
    contract WillSelfDestruct {
        constructor() payable {
        }
        function shovesEther(address _EthernautContractAddress) public {
            selfdestruct(payable(_EthernautContractAddress));
    
        }
    }
    //code is self explanatory. 
    ```
    
* Compile and Deploy this with '1000000' Wei, or any value for that matter.
    
* In the "Deployed Contract" section in the "Deploy and run" section, put your instance address as the `_EthernautContractAddress`
    
* Run the shovesEther function.
    
* Submit Instance.
    

Force✅