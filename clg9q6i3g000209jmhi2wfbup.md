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

As you can see from the above code, to clear the level. Our new contract must return 42 and the contract should be at most 10 opcodes. So the only way to solve this is to create a smart contract that only returns `42`.

As we have seen in a previous challenge, there are two types of bytecodes.

1. Creation bytecode: It is responsible for creating and preparing the contract.
    
2. Runtime bytecode: It is the code/logic of the contract.
    

To learn more in-depth about bytecodes and opcodes. [Read this](https://medium.com/@blockchain101/solidity-bytecode-and-opcode-basics-672e9b1a88c2#:~:text=Opcodes%20are%20the%20low%20level,opcodes%20and%20their%20hexadecimal%20values.).

### Analysing Opcodes

Let's figure out, the steps needed to create our runtime and creation bytecode. If you have read the above-linked article, you must know that this is how EVM works.

We have the solidity (high level) ---&gt; opcodes (low-level) ----&gt; bytecode (machine level, hex).

So, we would need to create our runtime bytecode and the creation bytecode. Let's first deal with the runtime bytecode. Look at this [opcode chart](https://github.com/crytic/evm-opcodes) for reference. Also, have a look at the LiFO principle before looking below.

1. We need to push and store the value 42 in the memory.
    
    ```solidity
    //8. Will push 2a(i.e. 42) on stack. 0x60 is bytecode for PUSH1. 
    0x602a
    //9. Will push a random memory slot 90 in the stack.
    0x6090
    //10. Will store (value p = 0x2a at position v = 0x90) to memory, 0x52 is the bytecode for MSTORE.
    0x52
    ```
    
2. Now, we need the contract to return this stored value (i.e. 42)
    
    ```solidity
    //11. 0x60 is for PUSH1 and `20` computes the 32 bytes hash which is essentially the size of v in stack. 
    0x6020
    //12. Value was stored in the 0x90 slot. 
    0x6090
    //13. Returns value at the slot 0x90 (42) which is of size 32 bytes. 
    0xf3
    ```
    

Hence, our final runtime bytecode will be: `0x602a60905260206090f3`

After that, let's have a look at the creation bytecode, this would be responsible for loading our runtime opcodes in memory and returning it to the EVM.

`COPYCODE` opcode is used to copy the runtime opcode. It takes in three parameters.

* The destination position, we will keep this as `0x00` it's the first position in memory.
    
* Current position of runtime code, we do not know this yet.
    
* Size of the runtime code. `0x602a60905260206090f3` which is 10 bytes.
    

1. ```solidity
    //1. Pushes the size of opcode which is 10 bytes (0a)
    0x600a
    //2. pushes the position of runtime code, but it is not known yet. 
    0x60--
    //3. pushes the desitnation position in memory at 0x00
    0x6000
    //4. Calls the copy code with all arguments
    0x39
    //Now we need to `return` this via 0xf3, which requires position and size, so let's do this.
    //5. Pushes the size of opcode which is 10 bytes (0a)
    0x600a
    //6. Pushes the desitnation position in memory at 0x00
    0x6000
    //7.Returns value at the slot 0x00 which is of size 10 bytes. 
    0xf3
    ```
    

Hence the initialization opcode is `0x600a60--600039600a6000f3`.

As we can see this is 12 bytes long, hence runtime opcode starts at index 12 which is `0x0c`. Thus the initialisation opcode is: `0x600a600c600039600a6000f3`

Thus, the final bytecode will be, creation + runtime. `0x600a600c600039600a6000f3602a60905260206090f3`

### Solution

1. Create a new level instance and fire-up the console.
    
2. ```javascript
    final_bytecode = '0x600a600c600039600a6000f3602a60905260206090f3' //we made this final bytecode. 
    txn = await web3.eth.sendTransaction({from: player, data: bytecode}) //sends this bytecode as transaction. 
    solverAddress = txn.contractAddress //saves its address.
    await contract.setSolver(solverAddress) //passes that address from the level instance. 
    ```
    
3. Submit Instance.
    

Magic Number🪄