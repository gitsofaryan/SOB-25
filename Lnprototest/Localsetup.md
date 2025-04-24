# LNprototest Competency Test Report

## Overview
This Gist documents my competency test for the **LNprototest Message Flow Visualizer** project as part of my Summer of Bitcoin (SoB) 2025 application. The test involved setting up and running `lnprototest` with LDK-Sample to simulate Lightning Network protocol interactions, focusing on BOLT compliance. Below, I outline the steps I followed, the results, the significance of this test, and insights into `lnprototest`’s workings, demonstrating my readiness to contribute to the project.

## Setup and Execution

### Steps Followed
I followed Prakhar Saxena’s instructions from the [lnprototest-sob-2025 Google Group](https://groups.google.com/d/msgid/lnprototest-sob-2025/CAG98NLvw%2BitjRgiqUCyZ2BHW-Jo48Bg%3D%3DrkbTsg5zXRpF_9YPg%40mail.gmail.com) to set up and run the test:

1. **Bitcoin Core Setup**: Ensured Bitcoin Core 23.0 was installed and added to my PATH.
2. **Cloning Repositories**: Created a `WORKDIR` directory and cloned `lnprototest` ([rustyrussell/lnprototest](https://github.com/rustyrussell/lnprototest)) and LDK-Sample ([Psycho-Pirate/ldk-sample](https://github.com/Psycho-Pirate/ldk-sample)).
   ```
   WORKDIR/
     - lnprototest/
     - ldk-sample/
   ```
3. **Build LDK-Sample**: Navigated to `ldk-sample` and built it using:
   ```
   cargo build
   ```
4. **Set PYTHONPATH**: Added the `lnprototest` directory to my Python path:
   ```
   export PYTHONPATH="$PYTHONPATH:/Users/maila/Desktop/SummerOfBitcoin/WORKDIR/lnprototest"
   ```
5. **Install Dependencies and Run Tests**: Inside `ldk-sample/Lnprototest_Testing`, installed dependencies and ran the tests:
   ```
   pip3 install poetry
   poetry install
   poetry run pytest /Users/maila/Desktop/SummerOfBitcoin/WORKDIR/lnprototest/tests --runner=ldk_lnprototest.Runner --log-cli-level=DEBUG
   ```

### Test Results
- **Outcome**: 29 tests passed, 17 were skipped, and 1 failed.
- **Failed Test**: `test_gossip_forget_channel_after_12_blocks` in `test_bolt7-01-channel_announcement-success.py` failed with an `EventError`:
  ```
  lnprototest.errors.EventError: `Got msg banned by {"event": "MustNotMsg", "file": "test_bolt7-01-channel_announcement-success.py", "pos": "50"}: 010063023be1b5b1f9fbb26fde890032bf4098fba6a78be75a8dd9deae332f6d20ec634806cc3477c41ca565c45089a8331beb3912fb896188b117d525ee17c85f637f2c0664c6205e3c9626b4b7618d18235c059f13579a41dd79ae99e1234d5e8709d295e77c3846b37c44edc7fc4bd1ab07b605c0d216973b6bb4d9dead54b98e083ccfe5a766dfe236446ddf52a4f89da5d3f3acc06851f2b0821223a069be02005f5cae7747451b8ae53709723c50fef2c7861c1b8b6bba1043e5bde56dbc39157a587d3495da5b953ba4a9f3a6ad8479a114e4bacbf4262dc257dd86f10db949e6e34e12ef95ead4ba5a9e38b86cd530920b7054914f1ee61ed4261783aa12000006226e46111a0b59caaf126043eb5bbf28c34f3a5e332a1fc7b2b73cf188910f0000670000010000...`
  ```
  This error occurred because the test expected no `channel_announcement` or `channel_update` messages after 12 blocks (per BOLT #7), but the LDK runner sent one unexpectedly.
- **Mentor Feedback**: Vincenzo Palazzo confirmed I successfully ran `lnprototest` and suggested reporting the failure as a bug to the `lnprototest` repository.

### Screenshot of Test Output
![Test Output Screenshot](https://github.com/user-attachments/assets/f7d629b0-267c-4288-9d9a-abcacc123482)



## Why This Test Matters
- **Protocol Compliance**: This test validates LDK’s adherence to BOLT #7, specifically the rule that nodes should forget a channel after a 12-block delay once its funding output is spent or reorganized. Identifying this failure highlights an implementation issue in LDK, which is critical for ensuring Lightning Network reliability.
- **Project Relevance**: The `LNprototest Message Flow Visualizer` aims to visualize message flows for protocol testing. Understanding how `lnprototest` interacts with LDK (e.g., sending `init` messages, handling gossip) directly informs my proposed React app’s API integration and visualization of message exchanges.
- **Debugging Skills**: My ability to set up the environment, run tests, and identify a failure demonstrates my readiness to debug and iterate on solutions, a key skill for the project (as noted in the requirements: “Be able to fail and reiterate on the solution”).

## Understanding `lnprototest` Internals
Based on the blog posts ([Part 1](https://blog.hedwig.sh/ln/lnprototest/lnprototest_intro_1ofn/), [Part 2](https://blog.hedwig.sh/ln/lnprototest/lnprototest_intro_2ofn/)):

- **Architecture**: `lnprototest` uses a `Runner` class to orchestrate interactions between a test client and a Lightning implementation (LDK in this case). It sends messages (e.g., `init`, `channel_announcement`) and checks responses against expected behavior defined in BOLT specs.
- **Test Workflow**: Tests like `test_gossip_forget_channel_after_12_blocks` simulate a channel lifecycle—funding, announcing, and closing—then verify gossip behavior. The `Runner` manages connections (`LDKConn`), sends messages, and tracks state (e.g., `must_not_events` to flag unexpected messages).
- **Failure Insight**: The failure indicates LDK didn’t forget the channel as expected, possibly due to a bug in its gossip handling or a misconfiguration in my setup (e.g., node syncing issues). This aligns with my project’s goal to visualize such discrepancies for easier debugging.

## Additional Insights for the Proposal
- **Experience with LDK**: Setting up LDK-Sample and running tests builds on my prior work with Bitcoin-related projects, like the block validation tests I tackled earlier, where I debugged mempool transaction issues. This experience ensures I can handle LDK interactions in my visualizer.
- **Python and Rust Proficiency**: I used Python to configure `lnprototest` and interacted with LDK (Rust-based) via the runner. While my focus was on Python, I’m prepared to dive deeper into Rust (e.g., reading LDK code) as needed, fulfilling the project’s skill requirements.
- **Iterative Approach**: The test failure and Vincenzo’s suggestion to report a bug mirror the iterative process I’ll use in the project—building, testing, and refining the visualizer based on feedback, much like my past work on Gitingest and Talk2Code.
- **Next Steps**: I’ll open an issue on `rustyrussell/lnprototest` with the test output and logs, contributing to the project even before SoB begins. This proactive step shows my commitment to improving `lnprototest`.

## Conclusion
This competency test confirms my ability to set up and run `lnprototest` with LDK, understand its internals, and identify implementation issues. It directly supports my proposal by showcasing my technical skills (Git, Python, debugging) and my understanding of Lightning Network protocols, preparing me to build an effective message flow visualizer. I’m excited to contribute to `lnprototest` and learn from Vincenzo and the community during SoB 2025!
