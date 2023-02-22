# Ethernaut - Walkthrough for Noobs -2 - Fallout

Okay, so this is an easy one. I will give you all the information to solve the challenge yourself and in the end, give out a solution for the ones who are stuck.

> Objectives:
> 
> Claim ownership of the contract.

### Background

A `constructor` in solidity is a special function, which is executed only once when the contract is created. It is essentially used to set the initial state of a contract and initialize the state variables. Also, the bytecode deployed on the network does not contain the constructor code.

You can learn more about constructors [here](https://medium.com/coinmonks/solidity-tutorial-all-about-constructors-46a10610336).

In the latest versions of Solidity, the constructors are declared as shown:

```solidity
pragma solidity 0.8.17;
contract exampleNew {
    constructor() { }
}
```

However, in earlier, now deprecated versions of solidity, constructors were declared as functions with the same name as the contract name.

```solidity
pragma solidity 0.4.0;
contract exampleOld {
    function exampleOld() public { } //this behaves like a constructor
}
```

### Code of Importance

```solidity
  /* constructor */
function Fal1out() public payable {
  owner = msg.sender;
  allocations[owner] = msg.value;
}
```

We can observe that if you call this function, you become the owner, but we can't call this since it is a constructor right?!

Wrong!

There are two problems here:

1. The name of the function != the name of the contract. The name of the contract is **Fal1out** and not **Fallout**.
    
2. Even if we correct the name of the function to Fallout, it still won't be a function since this syntax was only valid till Solidity &lt;= 0.4.21.
    

Since we know, that this is a normal function. We can call it externally and claim ownership.

### Solution

* Generate a new level instance and open up the console.
    
* Call the `Fal1out()` function, by typing this in your console.
    
* ```solidity
      await contract.Fal1out()
    ```
    
* You will have become the owner of the contract.
    
* You can submit the instance now.
    

Fallout ✅