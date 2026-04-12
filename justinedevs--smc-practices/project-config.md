---
trigger: always_on
description: Solidity Smart Contract Clean Practices and Project Safety
---


# Solidity Code Implementation Techniques

- Use descriptive, intention-revealing names for variables, functions, and contracts.
- Avoid vague or misleading names to improve readability, auditability, and maintainability.
- Each function, contract, or library should have one clear, focused purpose (SRP).
- Express code intent clearly to minimize comments; document "why" decisions when necessary.
- Replace magic numbers and strings with named `constant` or `immutable` variables.
- Organize project using modules and interfaces; split logic into reusable components with imports.
- Favor code simplicity and clarity; avoid overly clever code constructs or gas-cost tradeoffs that risk security.
- Implement centralized and consistent error handling using `require`, `assert`, and custom error types.
- Follow Checks-Effects-Interactions pattern for state changes involving external calls to avoid reentrancy.
- Avoid arithmetic overflow/underflow; use built-in checked math or SafeMath for older Solidity versions.
- Use access control modifiers (`onlyOwner`, role-based access) and OpenZeppelin libraries for sensitive actions.
- Write and maintain robust unit/integration tests before deploying contracts; use TDD methodology.
- Implement differential and fuzz testing using tools like Echidna, Foundry for property-based testing.
- Conduct testnet simulation with mainnet forking to replicate real-world interactions and catch edge cases.
- Perform gas and economic analysis; record and optimize for realistic gas limits with documented benchmarks.
- Prevent duplications by abstracting repeated logic into reusable libraries and utility contracts.
- Employ static analysis tools like Slither, MythX for vulnerability checks; resolve all high/critical issues before mainnet deployment.
- Regularly refactor code for simplicity and reduced technical debt.
- Do not use emoji, non-standard Unicode, or confusing ASCII art in contract code or comments.
- Use Solidity latest stable version unless required otherwise for compatibility.

# Avoiding Destructive Operations and Project Safety

- Restrict permissions for deployment, upgrade, and critical contract interactions; whitelist trusted addresses only.
- Use robust ownership and admin-control frameworks (Multi-sig, Timelock, etc.) for upgradeable contracts.
- Implement automated role management with on-chain governance (DAO patterns) for permission escalation/delegation.
- Integrate updatable allowlist mechanisms managed by contract owners or on-chain voting for access control.
- Do not implement code capable of self-destructing essential contracts (`selfdestruct`) or erasing core data.
- Log and audit all sensitive administrative operations; events for changes and access attempts.
- Enable contract pausing and emergency stop mechanisms (e.g., `Pausable`) for incident response.
- Back up source code and deployment scripts using version control (Git) and off-chain secure storage.
- Do not deploy raw test or development contracts to mainnet; use safe release and auditing processes.
- Implement automated CI pipelines for build, test, and security validation steps; validate critical actions in pre-commit/checks.
- Use comprehensive audit trails for future project reviews.

# Disaster Recovery and Incident Response

- Create clear disaster recovery playbooks for key compromise scenarios (emergency upgrade/disable, rotating roles).
- Document process for contract shutdown and post-mortem disclosure procedures tailored to on-chain context.
- Implement secure upgrade paths with transparent process for implementation swaps and pausing during upgrades.
- Require user/community approval before redeployment of critical contracts.
- Establish clear communication protocols for security incidents and emergency responses.
- Maintain emergency contact lists and escalation procedures for different severity levels.

# General Project Safety, Collaboration & Management

- Define clear roles, responsibilities, and access controls for all contributors, reviewers, and owners.
- Maintain high-quality documentation for contracts, ABI, deployment steps, and governance processes.
- Communicate security incidents, risk factors, and fixes effectively with the whole team.
- Foster regular code reviews, multi-party audits, and leverage community feedback where appropriate.
- Track high-impact bugs, confirmed vulnerabilities, upgrades, and rationale for all major changes.
- Use effective collaboration and communication platforms (Slack, Discord, etc.) to streamline development and incident response.
- Practice continuous improvement: conduct post-mortem/root-cause analysis after incidents and deployments.

# Bug Bounty and Community Engagement

- Implement standing public bug bounty programs (e.g., Immunefi) for incentivizing white-hat security reviews.
- Establish clear bug bounty scope, reward tiers, and disclosure timelines.
- Create channels for community security researchers to report vulnerabilities responsibly.
- Maintain public security contact information and clear reporting procedures.
- Recognize and reward community contributions to project security.

# Dependency Management and Supply Chain Security

- Snapshot and review all external contract/library dependencies in the build pipeline.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JustineDevs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
