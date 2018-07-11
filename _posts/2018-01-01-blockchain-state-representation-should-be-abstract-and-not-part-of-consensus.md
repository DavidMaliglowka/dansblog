---
layout: post
title: 'Blockchain State Representation should be Abstract and not part of Consensus'
date: 2018-01-01
author: Dan Larimer
cover: ''
tags: blockchain steem bitshares ethereum
---
![](https://steemitimages.com/0x0/https://cdn-images-1.medium.com/max/800/1*tTDkRnNDVk6i4vIgC_X2qw.jpeg)

In my last article I argued that blockchains should be designed like multiplayer games where all that is necessary is to synchronize user inputs in order to reach consensus on game state. Today I would like to make the argument that the exact form the game state takes is irrelevant so long as all players agree on the validity of new game inputs. Stated another way, I am going to argue that there is no need to reach consensus on a canonical global state that can be represented by a hash.

The reason I am writing this is because many people in the blockchain ecosystem consider this controversial. Ethereum is the primary example of a message-based (input based) protocol that doesn't use UTXO but does require all block validators to agree on the precise hash of the canonical state. I consider this requirement superfluous.

Any developer will tell you that the more constraints (requirements) you place on a solution, the more difficult it becomes to optimize the solution. If you over-specify your problem then you force yourself into design patterns that are less flexible and more computationally intensive. In the worst case, an over constrained problem is not solvable. This is why it is important to make sure every requirement actually adds value to the resulting product.

Why Does Ethereum maintain a global state hash?
-----------------------------------------------

The primary reason for maintaining a global state hash is for the purpose of generating proofs. If you have the hash of the global state and the state is organized in a merkle tree-like structure, then with a 1kb proof you can show that at a particular point in time a certain property had a particular value. For example, you could prove that last Monday your balance was 100 ETH or that 15 seconds ago your balance was 20 ETH.

At first glance that sounds wonderful, but what you cannot do is prove that *right now* your balance is greater than 10 ETH. The value of past states is meaningless when every pending transaction has the power to change the current state. Sophisticated developers would design around this by implementing time-locks into the state such that if you prove that your balance was 20 ETH and 30 seconds ago and that it was locked in a state that would prevent it from changing for 1 minute, then you can prove that "right now" it is greater than 10 ETH.

The problem with the time-lock design is it makes the assumption that the code will behave in such a way that will enforce that constraint on possible future values of the balance. This means that in addition to proving state, you must also prove the behavior of the code.

The secondary reason for maintaining a global state hash is to detect software bugs. In this context, a software bug is defined as non-deterministic behavior. A global state hash will not detect logic bugs of the kind that resulted in the failure of the DAO. In the event non-deterministic behavior is detected, the network will split into two or more fragments depending upon the nature of the non-determinism. In the best case, there exists a clear majority and the network continues advancing while the minority attempt to figure out why their machines do not match the majority. In the worst case there is no clear majority, just a largest minority. In no case does having a state hash enable a minority node to automatically correct itself and join the majority fork.

What happens if you don't have a global state hash?
---------------------------------------------------

Blockchains like Steem and BitShares opt to not generate nor require all nodes to agree on a global state hash. This means that it isn't possible to prove anything about blockchain state from block headers and a merkle proof alone. All Steem and BitShares can prove is that at a particular time a particular transaction was valid. Any transaction that was invalid would not be part of the blockchain.

If two peers suffer from a determinism bug then it will go undetected until one of the peers allows a transaction that is rejected by another peer. This could happen days, months, or years after the original divergence in the state. When it does happen the result is the same as what happens on Ethereum; minority peers are forced to figure out why they disagree with the majority and then fix the bug.

What we can conclude from this is that the benefit of a state hash is to fail-early in the event of non-deterministic behavior.

Sources of Nondeterministic Behavior
------------------------------------

If we want to enforce a design constraint around detecting non-deterministic behavior, then it is important that we understand how these things happen in the first place. Generally they boil down to the following broad categories:

1.  Reference to external state such as time, uninitialized memory, or a random number generator
2.  Differences in compiler optimization
3.  Differences in CPU architecture
4.  Differences in dynamically linked libraries
5.  Cosmic Rays and Hardware Bugs

Assuming all machines run the software in an identical environment and that cosmic rays or hardware bugs only impact individual nodes, then the only remaining sources for non-determinism are structural programming bugs.

Alternative means of Detecting Programming Bugs
-----------------------------------------------

Just because the hash of the state isn't part of the official consensus does not mean that peers cannot calculate a hash of their state for the purpose of detecting bugs early. In theory two peers running the same code should produce the same state and therefore if they hash their state it should produce the same value. If they do this periodically, then they would be able to catch a bug in determinism before it results in different peers drawing different conclusions on the validity of a particular transaction.

The key thing that makes performing checks outside consensus different from requiring these same checks as part of consensus is that it allows peers to completely change how they represent state without invalidating the global consensus. Two different peers can have completely different representations of state so long as they agree on which transactions are valid and invalid.

Debugging Code should not be part of Production
-----------------------------------------------

Developers everywhere know that before you ship code you strip out all of your performance monitoring and debugging tools. This developer instrumentation hurts performance and ultimately can interfere with meeting the real-time performance requirements of the product. In the case including hashes as part of the global consensus, it forces developers to exactly reproduce all historic bugs even if those bugs had no visible impact on the behavior of the platform. An example of a "bug" that does not impact behavior is switching from an array to a linked list. In this case the meaning of the data structure is the same, but the performance characteristics are vastly different.

Mathematical Generalization of Blockchain State
-----------------------------------------------

I normally avoid using math to prove things because I find that the vast majority of people cannot follow the logic. Today I am going to provide some math to generate a proof that blockchain behavior can be fully abstracted with different representations of the state while being mathematically equivalent.

Imagine there exists a function over all possible inputs `f(i)` such that it returned true if the input was valid and false if the input was invalid. Let . `I` be the set of all possible inputs and let `i` represent a specific input from that set. A blockchain such as Steem or BitShares can be defined such that `f(i)` returns `f(i)` if the input is invalid and returns `f2(i)` if the input is valid. `f2(i)` is a new function that behaves in a similar way to `f(i)` and which should be passed the next input. We could say that if the retuned function is the same as the original function then it returned `false`, else it returned `true`.

Two functions are considered identical if for each `i` in `I` they produce the same output. From a logical perspective, the internal implementation of `f(i)` is a black box. Therefore, the most abstract possible representation of state is as a mapping from `i` to `true` or `false`. With every input a new function is returned with a new mapping for every possible input.

It should be obvious that representing state this way is not practical, even though it is mathematically accurate. It isn't practical because the size of the input set `I` is effectively infinite. To overcome this problem we can use algorithmic compression techniques. If for example all ODD numbers are invalid and all EVEN numbers are valid, then we could express `f(i)` as `i % 2 == 0`. We could also express it as `i&1 == 0`. In both cases the implementation of `f(i)` is different, but both implementations are mathematically equivalent.

We can therefore conclude that it doesn't matter how the function is implemented so long as all parties have an implementation that produces the same true/false output for every possible input. We can also say that if blockchain validation is performed by checking the result of `f(i)` for each transaction, then all peers with equivalent functions will remain in consensus.

Impossible to Prove two functions Equal
---------------------------------------

The critics will quickly point out that the mathematical proof is meaningless in practice because it is impossible to prove that two different implementations of `f(i)` produce the same output for all input `i`. Anything beyond simple mathematical functions becomes impossible to prove. Ethereum attempts to prove they are equal by generating a hash of the state contained within `f(i)` and then asserting that given identical states and identical logic the two functions are equal.

Here is the rub, Ethereum this still requires proof that the logic is equal and free of determinism bugs. It is a circular proof! If we assert the code is free of determinism bugs then we can assert the states will be equal; however, if we cannot prove the code is free of determinism bugs then proving the state is equal does not actually help us prove that the two functions are equal. You can prove the presence of a bug, but not the absence of one. A difference in state hash may not matter if it is combined with an offsetting difference in logic.

Given that proving states to be equal by hash comparison does not actually prove consensus where consensus is defined as all parties agreeing on the validity or invalidity of all possible inputs, then what good is it? This is especially true in the case of multiple implementations in different languages. Proving you produced the same state is a good start, but doesn't actually prove anything as can be demonstrated by past Ethereum forks.

Why does any of this matter?
----------------------------

This matters because in an attempt to prove the impossible, blockchain engineers are over constraining the consensus process in ways that prevent blockchains from scaling in ways required by real world applications. Developers are forced into canonical representations and validators are forced into performing computations that are redundant for deterministic code.

This matters because proofing historical states is far less useful than proving the current state. It matters because building scalable blockchains that are not overly brittle is critical to the general adoption of blockchain technology.

Conclusion
----------

Blockchains should not make any particular expression of state an explicit part of their consensus algorithm. Doing so will only hurt performance while making software brittle and bug fixes more complex to retroactively apply. If developers wish to check for bugs, they can still run a version of the code that generates and compares hashes. If anyone cares about historical proofs, they can still be generated and used by the parties running the developer-build of the software.

Given the benefits of having a state hash can still be achieved without including it in consensus, and that including a state hash in consensus comes at a high price, there is no compelling reason for blockchain developers to make state hashes part of consensus.
