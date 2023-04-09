---
title: "Ethernaut - Walkthrough for Noobs - 18 - Magic Number"
datePublished: Sun Apr 09 2023 18:15:32 GMT+0000 (Coordinated Universal Time)
cuid: clg9q6i3g000209jmhi2wfbup
slug: ethernaut-walkthrough-for-noobs-18-magic-number
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680811771527/e31f24e1-f78d-4f37-a546-f290814eb03c.jpeg
tags: ethereum, solidity, web3, smart-contracts, 2articles1week

---

This one is tougher than usual and needs some knowledge of the EVM but no worries, I will explain everything from the ground up. In this challenge, we need to create a contract that is smaller than 10 bytes and returns a number 42 (psst. [Read this](https://en.wikipedia.org/wiki/The_Hitchhiker%27s_Guide_to_the_Galaxy)) when a function is called. Let's get into it.

> To solve this level, you only need to provide the Ethernaut with a `Solver`, a contract that responds to `whatIsTheMeaningOfLife()` with the right number.
> 
> Easy right? Well... there's a catch.
> 
> The solver's code needs to be really tiny. Really reaaaaaallly tiny. Like freakin' really really itty-bitty tiny: 10 opcodes at most.

### Background

Number 42 is commented out in the Ethernaut code, it becomes pretty obvious that we need to return 42 when the function `whatIsTheMeaningOfLife()` is called.

It is evident in the factory contract for this level. [Read here](https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/contracts/levels/MagicNumFactory.sol). This is the contract that checks our solution and creates a new level instance. Let's see what the contract requires.

```solidity

  function validateInstance(address payable _instance, address) override public view returns (bool) {

    // Retrieve the instance.
    MagicNum instance = MagicNum(_instance);

    // Retrieve the solver from the instance.
    Solver solver = Solver(instance.solver());
    
    // Query the solver for the magic number.
    bytes32 magic = solver.whatIsTheMeaningOfLife(); 
    if(magic != 0x000000000000000000000000000000000000000000000000000000000000002a) //32 bytes hex conversion of the number 42. 

return false;
    
    // Require the solver to have at most 10 opcodes.
    uint256 size;
    assembly {
      size := extcodesize(solver)
    }
    if(size > 10) return false;
    
    return true;
  }
}
```

As you can see from the above code, to clear the level. Our new contract must return 42 and the contract should be atmost 10 opcodes. So the only way to solve this is to