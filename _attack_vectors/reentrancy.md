---
layout: default
title: "Reentrancy"
---

# Reentrancy

| How attackers exploit a function before it's finished.

## What is a Reentrancy

Reentrancy is arguably the most infamous vulnerability in smart contract history. It occurs when a function makes an external call to an untrusted contract before it finalizes its own state changes. This allows an attacker to repeatedly call back into the original function, or other functions, exploiting the incomplete state to bypass security checks and drain funds.

The core problem is that certain external calls, especially those transferring value, cede control of the execution flow to the receiving address. If that address is a malicious contract, it can use its receive() or fallback() function (or a token callback) to hijack the execution and re-enter the calling contract. This creates a recursive loop that executes logic multiple times against a stale state.

### Real-World Impact: The DAO Hack

In June 2016, "The DAO," a pioneering decentralized venture fund, was exploited via a classic reentrancy attack. The attacker repeatedly called the withdraw function. Each time, the contract sent ETH to the attacker before updating their internal balance ledger. The attacker's contract re-entered the withdraw function again and again, draining 3.6 million ETH (worth ~$70 million at the time) and triggering a contentious hard fork of the Ethereum network that created Ethereum Classic.

## How the Attack Works

The vulnerability can manifest in several ways, from simple single-function exploits to complex cross-contract interactions.

1. **Classic  Reentrancy**

This is the original form. A single function updates a user's balance after sending them funds.

```sol
mapping(address => uint) public balances;

function withdraw() external {
    uint amount = balances[msg.sender];
    require(amount > 0);

    // VULNERABILITY: External call is made BEFORE state is updated.
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed.");

    // State update happens too late.
    balances[msg.sender] = 0;
}
```

**Attacker's Playbook**: The attacker's contract calls `withdraw()`. Its receive() function is triggered by the .call{value: amount}. Inside receive(), it calls `withdraw()` again. Because the line balances[msg.sender] = 0; has not yet been reached, the require check passes, and another transfer is sent. This loop continues until the contract is drained.

2. **Cross-Function Reentrancy**

A more subtle attack where the re-entrant call targets a different function within the same contract. A reentrancy guard on the withdraw function is useless if another function can manipulate the same state.

```sol
function withdraw() external nonReentrant { // Guard is on the wrong function
    uint amount = balances[msg.sender];
    // ... external call that re-enters ...
    balances[msg.sender] = 0;
}

function transfer(address to, uint amount) external {
    // Attacker re-enters here to move their "ghost" balance
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount;
    balances[to] += amount;
}
```


Attacker's Playbook: The attacker calls `withdraw()`. The re-entrant call doesn't target `withdraw()` again (which would be blocked) but instead calls transfer(). Since the attacker's balance hasn't been zeroed out yet, they can transfer their "soon-to-be-deleted" balance to another account they control.

3.**Read-Only Reentrancy**

A non-fund-stealing but equally dangerous variant. An attacker tricks a contract into reading a stale or intermediate state during a transaction. This is common with on-chain oracles that read from DEX liquidity pools.

Attacker's Playbook: An attacker initiates a large swap on a Uniswap V2-style pool. They re-enter an oracle contract that relies on that pool for its price feed. The oracle reads the now-imbalanced pool reserves, calculates a wildly inaccurate price, and returns it. The attacker can then use this faulty price in another protocol or simply revert their initial transaction, leaving no trace of the manipulation.

## Common Reentrancy Triggers

Reentrancy is not limited to low-level .call with ETH. Any external call that can trigger a callback to a user-controlled contract is a potential vector. Be vigilant with:

- **Low-level call()**: The classic vector for sending ETH.

- **ERC721 safeMint & safeTransferFrom**: These call onERC721Received on the recipient if it's a contract.

- **ERC777 transfer**: This standard has tokensToSend and tokensReceived hooks that can re-enter.

- **ERC1155 safeTransferFrom & safeBatchTransferFrom**: These call onERC1155Received or onERC1155BatchReceived.

- **Advanced/Custom ERCs**: Any token with a *AndCall pattern (e.g., ERC223, ERC677, ERC1363) or custom callbacks.

## How to Prevent It

1.**Checks-Effects-Interactions Pattern**

This is the single most important pattern for preventing reentrancy. Structure your functions in this order:

- **Checks**: Perform all validations first (require(balance > 0)).

- **Effects**: Write all state changes to your contract before the external call (balances[msg.sender] = 0).

- **Interactions**: Make the external call last (msg.sender.call{...}).

```sol
function withdraw() external {
    uint amount = balances[msg.sender];
    // CHECK
    require(amount > 0);
    // EFFECT
    balances[msg.sender] = 0;
    // INTERACTION
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed.");
}
```

2. **Use a Reentrancy Guard**

For an explicit layer of security, use OpenZeppelin's ReentrancyGuard. It provides a nonReentrant modifier that locks the contract during an external call, preventing any re-entrant calls to functions with the same modifier.

```sol
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract MyContract is ReentrancyGuard {
    function guardedWithdraw() external nonReentrant {
        // ... logic with external call ...
    }
}
```

_Note: This guard is highly effective but must be applied to all functions that share state to prevent Cross-Function Reentrancy._

3. **Use Pull-over-Push Payments**

Instead of the contract "pushing" funds to users, have users "pull" them. The withdraw function doesn't transfer ETH; it just credits an internal balance that the user can claim via a separate, isolated function call. This isolates the state change from the interaction.

4. **Understand Transient Storage (EIP-1153)**

Introduced in the Cancun upgrade, transient storage (TSTORE, TLOAD) provides a gas-efficient way to store data only for the duration of a single transaction. This is a modern, superior alternative for creating reentrancy locks, as it avoids costly SLOAD/SSTORE operations and automatically clears after the transaction, leaving no storage footprint. It's the future of robust, gas-efficient reentrancy protection.