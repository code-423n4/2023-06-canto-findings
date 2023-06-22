# Canto Audit Report

This audit report outlines the findings and observations for the Canto smart contract. The audit was conducted based on a review of the codebase, architecture, and mechanism review.

## Approach to Auditing the Canto Platform

### Phase 1: Documentation and Video Review
- Start with a comprehensive review of all documentation related to the Canto platform, including the whitepaper, API documentation, developer guides, user guides, and any other available resources.
- Watch any available walkthrough videos to gain a holistic understanding of the system. Pay close attention to any details about the platform's architecture, operation, and possible edge cases.
- Note down any potential areas of concern or unclear aspects for further investigation.

### Phase 2: Manual Code Inspection
- Once familiarized with the operation of the platform, start the process of manual code inspection.
- Review the codebase section by section, starting with the core smart contract logic. Pay particular attention to areas related to slippage handling, non-existing pools, and frontrunning prevention.
- Look for common vulnerabilities such as reentrancy, integer overflow/underflow, improper error handling, etc., while also ensuring the code conforms to best programming practices.
- Document any concerns or potential issues identified during this stage.

### Phase 3: Review of IBC Documentation and External Integration
- Thoroughly read the Inter-Blockchain Communication (IBC) protocol documentation. Ensure that Canto's implementation is in line with the standard, and note any deviations.
- Evaluate the handling of cross-chain transactions and the robustness of Canto's solution against possible external attacks or vulnerabilities.
- Examine how the platform integrates with external systems (e.g., oracles, other blockchains). Investigate how the platform handles communication errors or discrepancies in data from these systems.
- Note any potential weak points in the IBC and external integration implementation.

## Architecture Recommendations

Onboarding Module is well-designed with a proper sequence of operations. However, a few aspects should be noted:

1. Error Handling: The error handling in the onboarding middleware should be enhanced. Currently, even if the swap or conversion fails, it does not revert IBC transfer. The onboarding process is non-atomic, meaning that the asset transferred to the Canto network will still remain in the Canto network. A robust error handling system can greatly improve the system's reliability.

2. Middleware Ordering: The middleware ordering seems to be well thought out. However, the impact of the order on the performance and reliability of the transactions should be assessed in a real-time scenario.

3. Whitelisting Channels: The system works only when transactions are performed through a whitelisted channel. While this reduces risk, it could limit the flexibility of the system. Thus, further analysis of the trade-off between security and flexibility would be beneficial.

## Codebase Quality Analysis

The quality of the codebase appears to be of high quality and well-structured. A key aspect is the separation of concerns with different modules handling different aspects of the system, like onboarding, IBC transfers, and the Coinswap module.

The usage of Go modules also increases the maintainability and readability of the code. The code is well-commented, which can be beneficial for future developers.

However, it is recommended to use linters and static code analyzers to identify and rectify any potential security vulnerabilities, bugs, or code smells.

## Centralization Risks

There are risks related to the centralization of control in the system. A key risk area is the whitelisted channels for auto swap and convert. If the governance of the whitelisted channels is not properly decentralized, it may lead to centralization.

Moreover, the parameters like `enable_onboarding` and `auto_swap_threshold` could be manipulated by a centralized entity. A clear governance structure should be implemented to mitigate these risks.

## Mechanism Review

The system utilizes the Inter-Blockchain Communication (IBC) protocol for the transfer of assets and the onboarding module is used to convert these assets into Canto tokens and ERC20 tokens automatically.

The system uses an Automated Market Maker (AMM) model for swapping the assets. The Coinswap module handles the AMM functions. The overall mechanism seems to be well thought out.

However, the system should analyze the potential risks and vulnerabilities associated with the AMM model, such as impermanent loss and the risk of liquidity pools being drained.

## Handling of Slippage

Slippage is a significant issue in decentralized trading platforms, which occurs when the price changes during a transaction due to the liquidity's volatility. 

From the audit, it is observed that the Canto smart contract does not have explicit handling for slippage. This could lead to users experiencing unfavorable trade execution, especially for large orders that may move the price significantly. 

**Recommendations:**
1. Implement a slippage tolerance feature, where a user can specify a maximum acceptable price slippage for their trade. If the actual price slippage exceeds this tolerance, the transaction should be reverted.
2. To further mitigate slippage, consider using price oracles to provide accurate and timely pricing data, which can help in better trade execution.

## Handling of Non-Existing Pools

Currently, it appears that the system does not have specific handling for non-existing pools. This could potentially lead to failed transactions and unsatisfactory user experiences.

**Recommendations:**
1. Implement a mechanism to check the existence of a pool before initiating a transaction. If a pool does not exist, the transaction should be prevented or reverted, and the user should be notified appropriately.
2. Consider a feature that allows users or liquidity providers to create new pools when necessary, fostering a more flexible and user-friendly ecosystem.

## Systemic Risks

A systemic risk in this system is the dependence on the Gravity Bridge and Cosmos SDK. Any vulnerability or bug in the Gravity Bridge or Cosmos SDK would directly impact the Canto platform.

The system also heavily relies on the proper functioning of the IBC. Any issues in the IBC protocol could potentially cause serious implications for the system.

# Conclusion

The Canto platform, with its onboarding module, provides a user-friendly approach to convert assets into Canto and ERC20 tokens. However, the system should pay attention to error handling, potential centralization risks, and dependencies on external systems.

### Time spent:
20 hours