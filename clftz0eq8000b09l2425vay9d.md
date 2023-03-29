---
title: "Ethernaut - Walkthrough for Noobs - 14 - Gatekeeper Two"
datePublished: Wed Mar 29 2023 17:38:25 GMT+0000 (Coordinated Universal Time)
cuid: clftz0eq8000b09l2425vay9d
slug: ethernaut-walkthrough-for-noobs-14-gatekeeper-two
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680088064025/7e06d081-ff24-4a8b-9321-3c42bba97e28.jpeg
tags: ethereum, solidity, web3, smart-contracts, 2articles1week

---

This is similar to the previous challenge Gatekeeper One in the sense that, it also has three modifiers/gates that you need to pass. You will learn about `extcodesize()` and the bitwise `XOR` operator.

> This gatekeeper introduces a few new challenges. Register as an entrant to pass this level.
> 
> We need to pass all three gates.

### Gate 1

```solidity
modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
}
```

This should be pretty similar to [Gatekeeper One](https://adityabajaj.hashnode.dev/ethernaut-walkthrough-for-noobs-13-gatekeeper-one) and [**Telephone**](https://adityabajaj.hashnode.dev/ethernaut-walkthrough-for-noobs-4-telephone)**.** A word of advice or caution, if you are at Level 14 and you can't make sense of this. I suggest you do each and every level from the ground up again.

If we create an intermediary contract to call the GatekeeperTwo contract, we would pass this modifier.

### Gate 2

```solidity
modifier gateTwo() {
    uint x;
    assembly { x := extcodesize(caller()) }
    require(x == 0);
    _;
  }
```

We can leave solidity and shift to [inline assembly](https://docs.soliditylang.org/en/v0.8.16/assembly.html), for fine-grained control. This low-level language is called Yul.

> An inline assembly block is marked by `assembly { ... }`, where the code inside the curly braces is code in the [Yul](https://docs.soliditylang.org/en/v0.8.16/yul.html#yul) language.

`extcodesize` is an opcode that returns the size of the code on that address. If the size is larger than zero that address should be a contract.

`caller()` this is the address of the last call sender (except `delegatecall()` as seen in a previous challenge)

Here, `x` is used to store the size of the code on the caller's address. That is the contract we will be using to interact with GatekeeperTwo.

But, the problem is, this `x` is checked to be **zero** in the next line, which basically states that the caller should be an EOA. But again, the first gate ensures that we must interact with GatekeeperTwo with another contract.

How can both things be true at the same time?

For this to be true, a smart contract must have zero code, but how. Turns out there is a special case when this is true. There are two different bytecodes on Ethereum.

1. **Creation Bytecode**: this is required by Ethereum to create the contract. it includes the constructor logic, and constructor parameters of the smart contract, this generates runtime bytecode.
    
2. **Runtime Bytecode:** this describes the code in a smart contract and each and every function it executes. `extcodesize` will show the size of this.
    

So during contract initialization, it does not have any runtime bytecode, since the size of the contract at that point in time is zero. I think you understand what the idea here is. If we call the `enter()` function in the GatekeeperTwo contract from the constructor of our attack contract. We should be good to go.

*Note: Read more about runtime and creation bytecodes, pick your poison:* [*Easy Mode*](https://ethereum.stackexchange.com/questions/76334/what-is-the-difference-between-bytecode-init-code-deployed-bytecode-creation) *v/s* [*Nerd Mode*](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-ii-creation-vs-runtime-6b9d60ecb44c/)

### Gate 3

```solidity
modifier gateThree(bytes8 _gateKey) { //takes a 8 byte input
    require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
    _;
  }
```

Okay, this should also be fairly easy to understand. Let's break this down:

1. `(uint64(bytes8(keccak256(abi.encodePacked(msg.sender))))` : `msg.sender` is packed and encoded --&gt; returns bytes --&gt; which is hashed using keccak256 --&gt; low ordered 8 are taken from the hash ---&gt; which is casted to a `uint64`.
    
2. `^` : This stands for bitwise operator `XOR`. If bits at each position are the same it will output 0, else the output will be 1. [Read more](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_XOR).
    
    So if, `a ^ b == type(uint64).max` means that `a` must be the exact inverse of `b`.
    
    Also, we know that if `A XOR B = C` then `A XOR C == B` . This is not intuitive but logical. Think why this is true.
    

`type(uint64).max` : gives the maximum number that can be in `uint64`.

To pass the third gate, this should do the job.

```solidity
bytes8 myKey = bytes8(uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^ (type(uint64).max));

//if A XOR B == C  ----> A XOR C == B
```

### Solution

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

interface IGateKeeperTwo {
    function enter(bytes8 _gateKey) external returns (bool); //creating an interface
}

contract IWantIn {
    constructor() public {
        IGateKeeperTwo gatekeepertwo = IGateKeeperTwo('instance address without quotes'); //creates an instance of the interface. 
        bytes8 myKey = bytes8(uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^ type(uint64).max); //creates a Key that passes Gate 3. 
        gatekeepertwo.enter(myKey); //inputs that key in the enter function. 

    }
}
```

1. Open Remix, copy the code, compile, and deploy the contract.
    
2. Check if you are the entrant now in the console.
    
3. Submit Instance.
    

GateKeeperTwo**💂💂**