---
title: "Ethernaut - Walkthrough for Noobs - 12 - Privacy"
datePublished: Mon Mar 27 2023 15:03:20 GMT+0000 (Coordinated Universal Time)
cuid: clfqyl9nc000009mwexnd1cce
slug: ethernaut-walkthrough-for-noobs-12-privacy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679900866225/c554dc54-89c1-49ff-8577-a98464e64519.jpeg
tags: ethereum, solidity, smart-contracts, 2articles1week, ethernaut

---

This challenge is marked as a 4/5 difficulty level problem but would fail to meet your expectations. If you remember the Vault - Level 8, you pretty much understand the background required to get your hands dirty and solve the problem.

> Objective: The creator of this contract was careful enough to protect the sensitive areas of its storage.
> 
> Unlock this contract to beat the level.

### Background

The main concept behind the challenge is that nothing stored in the blockchain is truly private, the `private` variables are readable by others and also the data is stored in slots, similar to my article on vault: [Read here](https://adityabajaj.hashnode.dev/ethernaut-walkthrough-for-noobs-8-vault).

One more concept you should be aware of is downcasting and explicit conversions.

When we downcast a value you will lose information, since naturally, you cannot fit something in a box that is bigger than the box.

```solidity
bytes32 random32 = '0x6632a4653dd0709532bbaca095a70d32f28a9669466f9be1c9d9b4665b778e71'
//total 64 characters, so technically 66 including 0x

bytes16 random16 = bytes16(random32) //downcasting. 

//random16 --> '0x6632a4653dd0709532bbaca095a70d32f2' --> total 32 characters, so technically 34 including 0x. 
```

*Note: One question that a lot of beginners ask is, how a 32-byte hash can accommodate 64 digits. To do that you need to understand and learn hexadecimal and binary formats.*

*One byte is made up of 8 bits i.e. the range is 00000000 - 11111111 in binary, similarly, in hexadecimal, the range for one byte is 0x00 - 0xFF. One byte is represented in hex as a 2-character string. Therefore, a 32-byte hex string is 64 characters long.*

Before reading ahead, I hope you understand what is slot packing and what are private variables.

### Code of Importance

Okay, so we know that for statically-sized variables, solidity tries to pack multiple variables into a single slot (size 32 bytes) to optimize storage.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Privacy {

  bool public locked = true; //slot 0, takes 1 Byte, but cannot pack because of the next variable. 
  uint256 public ID = block.timestamp; //slot 1
  uint8 private flattening = 10; //slot 2
  uint8 private denomination = 255; //slot 2
  uint16 private awkwardness = uint16(block.timestamp); //slot 2
  bytes32[3] private data; //slot 3, 4, 5, 6 --> this is an array and each element takes one slot. 

//data[0] => slot 3 
//data[1] => slot 4
//data[2] => slot 5
//data[3] => slot 6

  constructor(bytes32[3] memory _data) { 
    data = _data;  //initialises data
  }
  
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2])); //key is in slot 5. 
    locked = false; //if we can fetch and enter the key, we are done, as locked will be false. 
  }

//this represents the famous rainbow kitty image, I guess.

  /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
  */
}
```

### Solutions

1. Create a new level instance and fire up the console.
    
2. ```javascript
    const data32 = await web3.eth.getStorageAt(contract.address, 5) //gets the data in slot 5
    const data16 = data32.slice(0,34) //downcasts to 16 bytes
    //the reason we take 34 instead of 32 is to, add the inital `0x`.
    await contract.unlock(data16) //unlocks
    
    await contract.locked() //check if this returns false. 
    ```
    
3. Submit Instance
    
    Privacy🔏