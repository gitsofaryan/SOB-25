
# P2Poolv2 NS3 Simulation

This repository contains an NS3-based simulation designed to analyze the P2Poolv2 sharechain, specifically focusing on how it handles "uncles" (a mechanism to reduce wasted work in blockchain systems). The simulation models a peer-to-peer (P2P) network ranging from a handful of nodes up to 10,000, simulating the production of shares (proof-of-work solutions) and factoring in network latency. Its primary purpose is to measure key performance metrics—such as **uncle rates** (shares that are valid but not part of the main chain) and **orphan rates** (shares that are discarded due to delays)—to evaluate how well the sharechain performs under various conditions. By doing so, it aims to provide insights that will help optimize parameter settings for the upcoming launch of P2Poolv2, ensuring the system is robust and scalable.

## Why It’s Important

The P2Poolv2 protocol leverages uncles to minimize the occurrence of orphan blocks, a concept inspired by blockchain research (e.g., Ethereum) to make mining fairer and more efficient. Orphan blocks occur when miners’ work is wasted due to network delays, which can disproportionately affect smaller miners. By simulating how the sharechain behaves with uncles under different network sizes and latency scenarios, this project helps developers understand the system’s scalability and fairness. For example, it can reveal whether high latency or a large number of nodes increases orphan rates excessively, allowing the team to tweak P2Poolv2’s design before it goes live. This is critical for ensuring miners of all sizes can participate effectively.

## Directory Structure

The repository is organized with a clean and logical structure to separate code, scripts, outputs, and documentation. Below is the full directory structure with explanations:

```
p2poolv2-ns3-simulation/
├── src/
│   └── p2pool-sim.cc          # Main C++ simulation script for NS3
├── scripts/
│   ├── run-sim.sh             # Bash script to download NS3, build, and run the simulation
│   └── ns3-sim.yml            # GitHub Actions workflow for automated simulation runs
├── results/
│   └── logs/                  # Directory for simulation logs and trace files
├── docs/
│   └── design.md              # Detailed explanation of the simulation design
├── README.md                  # This file: project overview, setup, and run instructions
└── LICENSE                    # MIT License for the project
```

### Directory Breakdown

- **`src/`**: Contains the core simulation code.
  - `p2pool-sim.cc`: The C++ script that defines the NS3 simulation. It models a P2P network, share generation, latency, and logs metrics like uncle and orphan rates.
- **`scripts/`**: Includes automation and helper scripts.
  - `run-sim.sh`: A Bash script that automates downloading NS3, building it, and running the simulation.
  - `ns3-sim.yml`: A GitHub Actions workflow file for running the simulation automatically on code pushes or pull requests.
- **`results/`**: Stores simulation outputs.
  - `logs/`: Directory where simulation logs and trace files (e.g., share propagation data) are saved.
- **`docs/`**: Holds additional documentation.
  - `design.md`: Explains the simulation’s design, including network topology, share generation, latency modeling, and metrics.
- **`README.md`**: This file, providing an overview, setup instructions, and how to run the simulation.
- **`LICENSE`**: The MIT License file, making the project open-source and reusable.

## What’s Included

This repository brings several components to the table, each serving a specific role:

- **Simulation Code**: `src/p2pool-sim.cc` is the heart of the project. Written in C++ for NS3, it allows users to configure variables like the number of nodes (up to 10,000), network latency (e.g., hardcoded or log-normal distribution), and share submission timing. It outputs statistics on shares, uncles, and orphans to `results/logs/`.
- **Helper Script**: `scripts/run-sim.sh` simplifies the process by downloading NS3 dynamically, setting it up, and executing the simulation. It’s designed for ease of use on Linux-based systems (e.g., WSL or native Linux).
- **Automation**: `scripts/ns3-sim.yml` defines a GitHub Actions workflow that automatically runs the simulation whenever code is pushed or a pull request is submitted. This ensures continuous testing without manual effort.

## Setup Instructions

Follow these steps to set up and run the simulation locally on a Linux-based system (e.g., Ubuntu, WSL):

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/gitsofaryan/p2poolv2-ns3-sim.git
   cd p2poolv2-ns3-sim
   ```

2. **Install Dependencies**:
   Ensure you have the required tools installed:
   ```bash
   sudo apt update
   sudo apt install -y build-essential gcc g++ python3 python3-dev cmake wget
   ```

3. **Run the Simulation**:
   Use the provided helper script to automate the process:
   ```bash
   bash scripts/run-sim.sh
   ```
   - This script:
     - Downloads NS3 (version 3.44) if not already present.
     - Builds NS3 with CMake.
     - Copies `p2pool-sim.cc` to the NS3 scratch directory and runs the simulation.
   - **Note**: The first run may take several minutes due to NS3 download and build time.

4. **View Results**:
   - Simulation outputs (logs and trace files) are saved in `results/logs/`.
   - Check these files for metrics like uncle rates, orphan rates, and share propagation details.

## How to Run It

- **Manual Run (Optional)**: If you prefer more control:
  1. Download NS3 manually from [nsnam.org](https://www.nsnam.org/releases/ns-allinone-3.44.tar.bz2).
  2. Extract it: `tar xjf ns-allinone-3.44.tar.bz2`.
  3. Build NS3:
     ```bash
     cd ns-allinone-3.44/ns-3.44
     mkdir build && cd build
     cmake -DNS3_EXAMPLES=ON -DNS3_TESTS=ON ..
     make -j4
     ```
  4. Copy the simulation code: `cp ../../../src/p2pool-sim.cc ../scratch/`.
  5. Run it: `./ns3 run scratch/p2pool-sim`.

- **Automated Testing**: Push changes to the repository, and the GitHub Actions workflow (`ns3-sim.yml`) will run the simulation automatically. Check the "Actions" tab on GitHub for results.

## Customizing the Simulation

To adapt the simulation for your needs:
- Edit `src/p2pool-sim.cc` to change:
  - Number of nodes (e.g., 10 to 10,000).
  - Network topology (e.g., mesh or random P2P).
  - Latency settings (e.g., fixed 100ms or log-normal distribution).
  - Share generation rate (e.g., every few seconds with a normal distribution).
- Update `results/logs/` paths in the code if needed.

## Example Output

After running, `results/logs/` might contain files like:
- `share-log.txt`: Timestamps of share generation, sending, and receiving.
- `metrics.txt`: Calculated uncle and orphan rates (e.g., "Uncle Rate: 5%, Orphan Rate: 2%").

