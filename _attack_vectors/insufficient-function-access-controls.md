---
layout: default
title: "Insufficient Function Access Controls"
rank: 5
amount_stolen: "$27,000,000"
incidents: 29
risk_level: "low"
year: 2024
---

# Insufficient Function Access Controls

| How leaving a door unlocked can let an attacker empty the entire building.

## What is Insufficient Function Access Control?

Insufficient Function Access Control is a vulnerability where a smart contract fails to properly restrict who can execute sensitive functions. It occurs when functions that should be protected—such as those that mint tokens, change critical parameters, or transfer ownership—are left public or external without the necessary permission checks. This allows any unauthorized user to call them, effectively giving them administrative privileges.

The root cause is a failure to implement or correctly apply the Principle of Least Privilege, where an address should only have the absolute minimum permissions required to perform its function. When this principle is ignored, the contract exposes an attack surface that can be exploited to drain funds, manipulate the protocol's state, or seize control.

### Real-World Impact: The Galaxy Fox Hack

The Galaxy Fox token airdrop was designed to use a Merkle tree to verify eligible participants. The protocol stored a Merkle root on-chain, and users could claim tokens by providing a valid proof. However, the setMerkleRoot function—the very function that defined the rules of the airdrop—had no access control. An attacker called this public function, replaced the legitimate root with a malicious one they controlled, and proceeded to mint enough GFOX tokens to drain the entire WETH-GFOX liquidity pool.

## How the Attack Works

This vulnerability manifests in several common patterns, from simple oversights to complex integration flaws.

1. **The Missing Modifier**

This is the most common form of the vulnerability. A developer writes a function and simply forgets to add a modifier like onlyOwner or onlyRole.

**The TSURU Exploit**: The TsuruWrapper contract had an onERC1155Received function, which was intended to be called by the ERC1155 token contract upon a deposit, minting new TSURU tokens in return. However, the function was public and lacked any check to verify that the caller was a legitimate token contract. An attacker called this function directly in a loop, minting an unlimited supply of TSURU tokens to themselves and draining the liquidity pool.

2. **Unprotected Administrative Functions**

This is a high-impact subset of the missing modifier pattern. Functions that control the core parameters of a protocol are the "keys to the kingdom" and must be strictly guarded.

Attacker's Playbook (Galaxy Fox):

* **Discovery**: The attacker scans the contract and finds a public function named setMerkleRoot(bytes32 newRoot).

* **Payload Crafting**: The attacker creates their own Merkle tree where their address is the only leaf, granting them the entire airdrop allocation. They compute the root of this malicious tree.

* **Execution**: They call setMerkleRoot() with their malicious root. The contract, lacking any access control, accepts the new root.

* **Profit**: The attacker now calls the claim() function with their valid proof for the new root, minting the tokens and draining the protocol's liquidity.

3. **Flawed Integration with Trusted Contracts**

Sometimes, access control exists but is based on a flawed assumption about an external protocol. A contract might correctly check that msg.sender is a trusted entity (like a Gelato keeper), but fails to account for who can trigger that trusted entity.

**The Alchemix Exploit**: Alchemix's harvest function could only be called by the Gelato Automate contract to prevent front-running. However, they integrated with a general-purpose version of Gelato that allowed anyone to trigger a task. An attacker exploited this by repeatedly triggering the harvest function via Gelato, enabling them to execute profitable sandwich attacks during the yield swaps, draining value from the protocol. The access control was technically present but practically useless.

4. **Inheritance-Based Oversights**

In complex systems with multiple layers of inheritance, a vulnerability can be inherited from a base contract. A developer might override a function but forget to apply the necessary access control that was expected in the child contract.

**The Maia DAO Exploit**: A CoreBranchRouter contract inherited a virtual function from a base contract. This function was not overridden with the proper caller restrictions in the final implementation. This oversight created a hidden backdoor, allowing an attacker to impersonate a trusted component of the system and drain funds.

## How to Prevent It

Preventing these vulnerabilities requires a defense-in-depth approach, combining secure coding patterns with rigorous verification.

1. **Adhere to the Principle of Least Privilege**

This is the foundational rule. By default, functions should be private or internal. Only make them public or external if absolutely necessary for the contract's public interface. Never expose more functionality than is required.

2. **Implement and Consistently Apply Access Control**

Use battle-tested access control patterns for all sensitive functions.

Simple Ownership: For contracts with a single administrator, use OpenZeppelin's Ownable.sol and apply the onlyOwner modifier.

```sol
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyContract is Ownable {
    function criticalFunction() public onlyOwner {
        // ... privileged logic ...
    }
}
```

- **Role-Based Access Control (RBAC)**: For more complex systems with multiple roles (e.g., MINTER_ROLE, PAUSER_ROLE), use OpenZeppelin's AccessControl.sol. This provides a more granular and flexible way to manage permissions.

3. **Secure Third-Party Integrations**

When interacting with external protocols (oracles, keepers, bridges), deeply understand their security model.

Verify if a trusted contract can be triggered by anyone.

If possible, use dedicated instances or versions of services that allow for msg.sender whitelisting.

Treat data coming from any external contract that can be influenced by an untrusted user as tainted.

4. **Rigorous Code Review and Testing**

- **Full Inheritance Review**: When auditing or reviewing code, inspect the entire inheritance chain. Do not assume base contracts are secure or that child contracts have correctly implemented all necessary overrides.

- **Static Analysis**: Use tools like Slither, which can automatically detect unprotected functions that are public or external and perform state changes.

- **Write Negative Tests**: Your test suite must include tests that attempt to call privileged functions from unauthorized accounts. These tests should be expected to revert. This confirms your access controls are working as intended.
