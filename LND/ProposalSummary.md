# Detailed Summary: LND Sweeper Fee Function Enhancement Proposal [Link](https://docs.google.com/document/d/1sMNvxOYuTt4RLnAd0lCMBrGLLSdLoVFw8sIbZRYBeQU/edit?usp=sharing)

## Project Overview
This proposal addresses fundamental inefficiencies in Lightning Network Daemon's (LND) transaction fee mechanism by implementing an adaptive fee calculation system. The current linear fee function creates suboptimal outcomes—overpaying for non-urgent transactions while underpaying for urgent ones. The proposed solution implements three specialized fee functions that adapt dynamically based on transaction characteristics, potentially reducing fees by approximately 30% while improving confirmation reliability.

## The Problem
LND's sweeper currently uses a one-size-fits-all linear fee function that fails to optimize for different transaction scenarios:

1. For transactions with long confirmation deadlines (>80 blocks), the current function allocates unnecessarily high fees, wasting users' resources.

2. For urgent transactions (<10 blocks), the linear function doesn't increase fees aggressively enough, leading to delayed confirmations.

3. The system lacks configurability for users with different priorities and doesn't account for transaction value or network congestion in its calculations.

## The Solution Explained

### Three-Tiered Fee Function System

1. **Cubic Delay Function** (for urgent transactions)
   - Applies to transactions needing confirmation within 10 blocks
   - Aggressively increases fees as deadline approaches using cubic scaling
   - Prioritizes confirmation speed over cost efficiency
   - Particularly effective during network congestion

2. **Enhanced Linear Function** (for medium-priority transactions)
   - Handles standard transactions with 10-80 block confirmation windows
   - Improves upon current implementation with optimized parameters
   - Balances cost efficiency with reasonable confirmation times

3. **Cubic Eager Function** (for low-priority transactions)
   - Designed for transactions with generous time constraints (>80 blocks)
   - Significantly reduces fees while ensuring eventual confirmation
   - Uses a cube root scaling to create a gentle fee curve
   - Achieves major savings (~30%) for non-time-sensitive operations

### Intelligent Selection Mechanism
The system will automatically select the appropriate fee function based on:

- **Transaction Urgency**: Measured by deadline_delta (blocks until required confirmation)
- **Value Density (ρ)**: Ratio of transaction value to weight, determining economic importance
- **Network Conditions**: Adapting to congestion levels dynamically
- **User Preferences**: Allowing manual overrides for specific requirements

### User Control Through RPC
The proposal includes a robust RPC interface allowing users to:

- Set default fee calculation preferences
- Override automatic selection for specific transactions
- Configure parameters for each fee function type
- Establish maximum fee budgets (e.g., fees cannot exceed 50% of transaction value)

### Budget Enforcement and Edge Cases
The implementation includes sophisticated handling of:

- Fee caps based on transaction value percentage
- Network congestion spikes
- Very small or very large transactions
- Extremely urgent deadlines
- Mempool replacement scenarios

## Expected Benefits

1. **Cost Savings**: Approximately 30% reduction in fees for non-urgent transactions
2. **Reliability**: 99.5% on-time confirmation rate for urgent transactions
3. **Adaptability**: Responsive adjustment to varying network conditions
4. **User Autonomy**: Fine-grained control over fee strategy
5. **Resource Efficiency**: Better allocation of fee resources across transaction types

## Testing and Validation Approach
The proposal outlines a comprehensive testing strategy using:

- Simulation with historical fee rate data
- Polar environment testing with realistic network conditions
- Benchmarking against the current implementation
- Edge case validation across multiple transaction scenarios

## Broader Impact
This enhancement represents a significant improvement to LND's on-chain efficiency. By optimizing fee allocation, it makes Lightning Network operations more economical, potentially increasing adoption and improving user experience. The proposed design aligns with Bitcoin's values of resource efficiency and user sovereignty.
