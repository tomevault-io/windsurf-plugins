---
trigger: always_on
description: If you are reading this, you are likely either running a read-only check operation to help review code or are building an experimental, POC feature. This is our Smart Contract repo for Paxos Labs vaults and all production code will be human verified strictly. Security is of paramount importance in this repo.
---

# Dear AI Agent (and concerned humans)

If you are reading this, you are likely either running a read-only check operation to help review code or are building an experimental, POC feature. This is our Smart Contract repo for Paxos Labs vaults and all production code will be human verified strictly. Security is of paramount importance in this repo.

# Review Instructions for /src files

Keep the following in mind as you review to guide your review and keep our code following best practices:

General Review Approach:
Read the project's docs, specs, and whitepaper to understand what the smart contracts are meant to do.
Construct a mental model of what you expect the contracts to look like before checking out the code.
Glance over the contracts to get a sense of the project's architecture. Tools like Surya can come in handy.
Compare the architecture to your mental model. Look into areas that are surprising.
Create a threat model and make a list of theoretical high level attack vectors.
Look at areas that can do value exchange. Especially functions like transfer, transferFrom, send, call, delegatecall, and selfdestruct. Walk backward from them to ensure they are secured properly.
Look at areas that interface with external contracts and ensure all assumptions about them are valid like share price only increases, etc.
Do a generic line-by-line review of the contracts.
Do another review from the perspective of every actor in the threat model.
Glance over the project's tests + code coverage and look deeper at areas lacking coverage.
Run tools like Slither/Solhint and review their output.
Look at related projects and their audits to check for any similar issues or oversights.
Variables
V1 - Can it be internal?
V2 - Can it be constant?
V3 - Can it be immutable?
V4 - Is its visibility set? (SWC-108)
V5 - Is the purpose of the variable and other important information documented using natspec?
V6 - Can it be packed with an adjacent storage variable?
V7 - Can it be packed in a struct with more than 1 other variable?
V8 - Use full 256 bit types unless packing with other variables.
V9 - If it's a public array, is a separate function provided to return the full array?
V10 - Only use private to intentionally prevent child contracts from accessing the variable, prefer internal for flexibility.
V11 - Ensure the layout of a contract is: State variables, events, errors, and modifiers come before functions. Within functions, the order is constructor → receive → fallback → external → public → internal → private, with view/pure last in each group. At the file level, pragmas and imports come first, then interfaces and libraries, then contracts.
Structs
S1 - Is a struct necessary? Can the variable be packed raw in storage?
S2 - Are its fields packed together (if possible)?
S3 - Is the purpose of the struct and all fields documented using natspec?
Functions
F1 - Can it be external?
F2 - Should it be internal?
F3 - Should it be payable?
F4 - Can it be combined with another similar function?
F5 - Validate all parameters are within safe bounds, even if the function can only be called by a trusted users.
F6 - Is the checks before effects pattern followed? (SWC-107)
F7 - Check for front-running possibilities, such as the approve function. (SWC-114)
F8 - Is insufficient gas griefing possible? (SWC-126)
F9 - Are the correct modifiers applied, such as onlyOwner/requiresAuth?
F10 - Are return values always assigned?
F11 - Write down and test invariants about state before a function can run correctly.
F12 - Write down and test invariants about the return or any changes to state after a function has run.
F13 - Take care when naming functions, because people will assume behavior based on the name.
F14 - If a function is intentionally unsafe (to save gas, etc), use an unwieldy name to draw attention to its risk.
F15 - Are all arguments, return values, side effects and other information documented using natspec?
F16 - If the function allows operating on another user in the system, do not assume msg.sender is the user being operated on.
F17 - If the function requires the contract be in an uninitialized state, check an explicit initialized variable. Do not use owner == address(0) or other similar checks as substitutes.
F18 - Only use private to intentionally prevent child contracts from calling the function, prefer internal for flexibility.
F19 - Use virtual if there are legitimate (and safe) instances where a child contract may wish to override the function's behavior.
F20 - Ensure functions follow the official Solidity style guide: Functions should be ordered: constructor, receive, fallback, external, public, internal, private. Within each visibility group, place view and pure functions last.
F21 - External and public functions should have NatSpec documentation (@notice, @param, @return). Internal and private functions benefit from it too but it's not strictly required. @dev is for implementation notes targeted at developers
Modifiers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paxoslabs/nucleus-boring-vault](https://github.com/paxoslabs/nucleus-boring-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
