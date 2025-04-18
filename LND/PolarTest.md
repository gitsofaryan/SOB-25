# Exploring Fee Function with Polar
![image](https://github.com/user-attachments/assets/5ea439e5-3408-43f1-aefd-3d66e99f0e0c)

## Introduction

This competency test examines the fee function implementation in the Lightning Network Daemon (LND) using Polar, a simulation tool for managing Lightning Network nodes in a controlled Regtest environment. My objective was to set up a test network, perform channel operations, and analyze how LND handles transaction fees during sweeping after a force-closed channel. This hands-on approach provided practical insights into LND’s fee management.

---

## Network Setup

I configured a test network in Polar with the following components:

- **Nodes**:
  - Two LND nodes (v0.18.4-beta): **Alice** and **Bob**.
  - One Bitcoin Core node for blockchain operations in Regtest mode.
- **Steps**:
  1. Launched Polar and initialized a network named "LND Fee Test."
  2. Added Alice, Bob, and the Bitcoin Core node, then started the network.

### Funding Process

To prepare for channel operations:

- Generated new Bitcoin addresses for Alice and Bob.
- Mined blocks to create mature coins, sending **1 BTC** to each node.
- Confirmed transactions by mining additional blocks (e.g., 101 blocks to mature coinbase outputs).

**Challenge Overcome**: Initial funding attempts failed due to immature outputs, resolved by mining sufficient confirmations.

---

## Channel Operations

### Opening the Channel

- **Action**: Opened a channel from Alice to Bob with a capacity of **50,000 satoshis**.
- **Confirmation**: Mined 6 blocks to activate the channel.
- **Result**: A fully operational channel, verified in Polar’s interface.

### Force-Closing the Channel

- **Action**: Force-closed the channel from Alice’s node.
- **Monitoring**: Used `lncli pendingchannels` to track the sweeping process.
- **Purpose**: Triggered LND’s sweeper to reclaim funds, enabling fee behavior analysis.

---

## Fee Function Analysis

LND (v0.18.4-beta) employs a **linear fee function**, increasing fees steadily as deadlines approach. Below are my findings:

### Key Observations

- **Initial Fee Rate**: Began at **10 sat/vB**, derived from Bitcoin Core’s fee estimator.
- **Maximum Fee Rate**: Rose to **250 sat/vB** for a 0.0005 BTC HTLC, limited by a budget of 50% of the HTLC value.
- **Behavior**: Fees start conservatively and scale linearly to ensure confirmation before timelock expiry.

### Configuration Experiments

I adjusted settings to explore fee flexibility:

- `bitcoin.conf_target=4`:
  - Increased initial fee to **15 sat/vB** for faster confirmations.
- `sweeper.budgetproportion=0.3`:
  - Reduced maximum fee to **150 sat/vB**, capping the budget at 30% of HTLC value.

These tweaks highlight LND’s adaptability to balance cost and speed.

---



## Conclusion

This simulation deepened my understanding of LND’s fee function and its practical implications. Key insights include:

- The linear fee mechanism’s role in managing transaction costs and timing.
- The impact of configuration adjustments on fee behavior.
- The potential for advanced fee strategies to enhance Lightning Network efficiency.

This experience has prepared me to contribute to LND development, particularly in optimizing fee management for LND Sweeper function enhancement.

---

