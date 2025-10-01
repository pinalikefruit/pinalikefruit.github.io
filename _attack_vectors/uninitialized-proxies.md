---
layout: default
title: "Uninitialized Proxies"
---
# Uninitialized Proxies

Implementing upgradability in your smart contract? This is the vulnerability behind a **$10 million bug bounty**. It’s one of the simplest mistakes to make and one of the most devastating.


## What is an Uninitialized Proxy

An Uninitialized Proxy is a vulnerability that occurs in upgradeable smart contracts using the proxy pattern. It allows an attacker to seize ownership and gain complete administrative control over the contract, often leading to a total loss of funds or rendering the protocol inoperable. 

The root cause is a failure to call the `initialize()` function on a proxy contract after deployment, leaving it in a state where anyone can claim it.

This vulnerability stems from a fundamental aspect of the proxy pattern: the separation of **state** and **logic**. The proxy contract holds the state, while an implementation contract holds the logic. Because a constructor only runs in the context of the contract being created, it only sets the state for the implementation contract itself, not the proxy.

### Real-World Impact: The Wormhole Bug

In _February 2022_, a whitehat hacker discovered a critical uninitialized proxy vulnerability in Wormhole's bridge contract on Ethereum. By calling the `initialize()` function on the core bridge contract, an attacker could have gained ownership, replaced the implementation with a malicious contract, and triggered `selfdestruct` function. This would have destroyed the logic contract, leaving the proxy pointing to an empty address and permanently stop the protocol.

The vulnerability was responsibly disclosed, and the hacker was awarded a [$10 million bug bounty](https://medium.com/immunefi/wormhole-uninitialized-proxy-bugfix-review-90250c41a43a), the largest in history, for preventing the exploit.

## How the Attack Works

In a proxy setup, you have two separated components:

* **The Proxy**: This is the contract users interact with. It holds all the assets and the contract state (like `owner`, `balances`, etc.). It contains very little logic other than to delegatecall to the implementation.

* **The Implementation**: This contract contains all the business logic (e.g., `transfer()`, `mint()`, etc.). It holds no state and no assets.

**The Role of delegatecall**

When a user sends a transaction to the Proxy, the Proxy uses the delegatecall opcode to execute the function logic from the implementation. The feature of delegatecall is that the implementation's code runs in the context of the calling contract (the Proxy). This means the Implementation's code directly modifies the Proxy's storage.


**Why the constructor fails**

A constructor runs only once when a contract is first deployed. When you deploy your `Implementation.sol`, its constructor runs only in the context of the `Implementation.sol` contract itself. It sets the owner variable in the implementation's storage, which is completely ignored and unused by the proxy. The Proxy's storage remains untouched and **uninitialized**.

**The `initialize()` Function Trap**

To solve this, upgradeable contracts use a regular function, typically named `initialize()`, to act as the constructor. This function must be called through the proxy after deployment. When this happens, delegatecall ensures the initialization logic runs in the proxy's context, correctly setting the owner in the proxy's storage.

The vulnerability occurs in the window between deploying the proxy and calling `initialize()`. If that call is forgotten, the `initialize()` function remains a publicly callable function that anyone can use to claim ownership.

## How to Prevent It

Prevention requires a combination of correct deployment procedures:

1. **Atomic Initialization**

    The step is to ensure the `initialize()` function is called atomically, ideally in the same transaction or deployment script that creates the proxy.

2. **Use `Initializable.sol`**

    Use OpenZeppelin's `Initializable.sol` contract. It provides an initializer modifier that ensures an initialize function can only be called once, effectively mimicking a constructor's one-time execution.

```js
    import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

    contract MyContract is Initializable {

        function initialize() public initializer {
        
        }
    }
```

    * **initializer modifier**: This locks the function after its first successful execution.

    * **Correctly Call Parent Initializers**: If your contract inherits from other upgradeable contracts (e.g., `OwnableUpgradeable`), your `initialize()` function must call the `__init` function for every inherited contract (e.g.,` __Ownable_init()`, `__Pausable_init()`). This ensures the entire inheritance chain is set up.

    * **Use onlyInitializing for base Contracts**: If you are writing a base contract that is meant to be inherited, use the `onlyInitializing` modifier on its `__init` function instead of the initializer modifier.

3. **Disable Initializers in the Implementation**

    To prevent anyone from calling initialize on the implementation contract itself, preventing it from being claimed or misused:

```js
    constructor() {
        _disableInitializers();
    }
```
4. **Avoid use**:

    * **Do Not Use `selfdestruct`**: Never include a function with `selfdestruct` in an implementation contract. An attacker gaining control of the implementation could trigger it, destroying the logic contract.

    * **Note on the Cancun Fork (EIP-6780)**: While `selfdestruct` has been changed on the Ethereum mainnet post-Cancun (it no longer deletes code/storage), it remains fully destructive on L2s and other chains where the change is not yet active. It should be avoided entirely.

    * **Do Not Use delegatecall inside an implementation contract**: The proxy already uses delegatecall to run the implementation's code. This creates a dangerous, unpredictable chain of calls and should be avoided.

5. **Verification: Post-Deployment Checks**

    Deployment scripts should always conclude by reading the state from the proxy contract to verify that critical state variables like the owner have been set correctly. Implement monitoring tools to provide a secondary level of verification that initialization has occurred properly.