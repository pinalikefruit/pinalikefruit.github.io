---
layout: default
title: "Supply Chain Attacks"
---

# Supply Chain Attacks

| When the hack happens before a single line of your code is deployed.

## What is a Supply Chain Attack?

A Supply Chain Attack in Web3 is a sophisticated exploit that targets the external dependencies, tools, and infrastructure that developers rely on to build and deploy smart contracts. Instead of attacking the contract's logic directly, an attacker compromises a critical component in the development pipeline—such as a code library, a frontend framework, or a deployment script—to inject malicious code.

This is one of the most insidious types of attacks because the vulnerability doesn't exist in the developer's own code. The project can be perfectly written, audited, and tested, yet still be vulnerable because it inherits a flaw from a trusted, third-party source. The attacker exploits the trust developers place in the tools they use every day.

### Real-World Impact: The Ledger Connect Kit Exploit

In December 2023, a widespread attack targeted numerous DeFi frontends, including those of SushiSwap, Zapper, and Revoke.cash. The attacker didn't hack any of these protocols' smart contracts. Instead, they compromised a single, widely used JavaScript library: Ledger's connect-kit.

The attacker gained access to the NPM account of a former Ledger employee and published a malicious version of the library. Any decentralized application (dApp) that used this library automatically pulled the compromised code into its frontend. When users visited these trusted dApps, the malicious code would pop up a fake wallet connection prompt, tricking them into signing transactions that drained their wallets. The dApps' own code was secure, but their supply chain was not.

## How the Attack Works

A Web3 supply chain attack can target any link in the chain from code creation to user interaction.

1. Compromised Code Libraries (NPM/Package Manager Attack)

This is the most common vector, as seen in the Ledger and CoW Swap exploits.

The Attack Vector: An attacker gains unauthorized access to a popular package on a repository like NPM (Node Package Manager). This can be done through phishing a developer, using stolen credentials, or exploiting weak account security (e.g., no 2FA).

The Payload: The attacker publishes a new, minor version of the package (e.g., from 1.1.7 to 1.1.8). This new version contains a small, obfuscated piece of malicious code designed to steal private keys, drain funds, or trick users into signing malicious transactions.

The Propagation: Most projects use versioning like ^1.1.7, which automatically pulls the latest minor update. When developers or CI/CD systems build the project, they unknowingly download and bundle the malicious code.

2. Malicious Code Injected via Browser Extensions

This attack targets the end-user's environment. A malicious browser extension can manipulate the content of a trusted dApp's website as it is rendered in the user's browser. It can change a destination address for a transfer, alter the amount, or replace the entire UI with a phishing site, all without the dApp's server or smart contracts ever being compromised.

3. Compromised Development Tools & Infrastructure

This is a deeper, more targeted attack on the developer's own environment.

Compiler/IDE Attack: A theoretical but highly dangerous attack where the compiler (e.g., a malicious version of solc) or an IDE extension (e.g., a compromised VS Code plugin) is modified to subtly alter the bytecode of a smart contract during compilation. The source code looks perfect, but the deployed bytecode contains a hidden backdoor.

DNS Hijacking: An attacker gains control of a project's DNS records and points the official domain (e.g., my-dapp.com) to a malicious server hosting a perfect clone of the site. Users who visit the familiar URL are interacting with a phishing site from the start.

## How to Prevent It

Preventing supply chain attacks requires a security mindset that extends beyond the smart contract to the entire development and operational lifecycle.

1. Lock Your Dependencies

This is the single most effective defense against package manager attacks. Do not allow your project to automatically pull the latest updates.

Use a Lockfile: Use package managers that generate a lockfile (e.g., package-lock.json for NPM, yarn.lock for Yarn). This file records the exact version of every dependency and sub-dependency used in a project. When another developer or a CI/CD pipeline builds the project, it will install the exact same versions, preventing a malicious update from being pulled automatically.

Commit Your Lockfile: Always commit your package-lock.json or yarn.lock file to your version control repository (e.g., Git).

2. Vet and Audit Your Dependencies

Treat your dependencies as part of your codebase.

Minimize Dependencies: Use as few external libraries as possible. Every new package is a new potential attack surface.

Review Updates Carefully: When you do need to update a package, review its changelog and code. Use tools that can scan for known vulnerabilities in your dependencies (e.g., npm audit, Snyk).

3. Secure Your Deployment Pipeline and Environment

Use Subresource Integrity (SRI): For frontend applications, when loading scripts from a third-party source or CDN, use SRI. This involves adding a cryptographic hash of the expected script to the <script> tag. The browser will only execute the script if its hash matches the one you provided, preventing a compromised CDN from serving you malicious code.

Strong Access Controls: Enforce mandatory 2FA on all critical accounts (NPM, GitHub, DNS providers). Use hardware keys for the highest level of security.

DNSSEC: Implement DNSSEC (Domain Name System Security Extensions) to cryptographically verify DNS responses, making it much harder for an attacker to successfully hijack your domain.

4. Educate Your Users

While you can't control your users' environments, you can educate them. Encourage the use of wallet simulation tools and hardware wallets. Promote best practices like bookmarking official sites and being wary of unexpected pop-ups or transaction requests. A vigilant user base is the last line of defense.

