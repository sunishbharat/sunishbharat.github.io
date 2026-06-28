---
title: Qt/C++ CAN Hardware Simulator
description: Replaced physical automotive development tools costing USD 5K-14K per developer seat, enabling parallelized overnight execution of 1,000+ test cases at zero cost.
---

# Qt/C++ CAN Hardware Simulator

!!! abstract "Summary"
    **Type**: Tooling project - automotive Head Unit development
    **Stack**: C++, Qt Framework, CAN Bus

    **Impact:**

    - Physical tool cost per developer seat eliminated: USD 5,000-14,000 → $0
    - 1,000+ automotive test cases parallelized and run overnight
    - Test cycle moved from hardware-dependent sequential runs to parallel CI execution

## Challenge

Development and testing of an automotive Head Unit required physical CAN bus hardware interfaces - proprietary tools costing between USD 5,000 and USD 14,000 per developer seat. This created two problems:

1. **Access bottleneck**: The number of developers who could run tests simultaneously was capped by hardware availability. Test execution was sequential and hardware-dependent.
2. **CI/CD incompatibility**: Automated overnight test runs were not feasible because they required physical hardware to be attached and available.

The project had a test suite of 1,000+ automotive test cases that needed to run reliably and frequently, but the physical hardware constraint made this impractical.

## Approach

Reverse-engineered the vehicle Head Unit's hardware communication behavior and built a software CAN simulator in Qt/C++ that faithfully replicated the hardware interface at the protocol level.

The simulator intercepted CAN frames at the software boundary, processed them through the same state machine logic the physical hardware would execute, and returned the expected responses. From the perspective of the test suite, the simulator was indistinguishable from the physical hardware.

Key aspects:

- **Protocol-level fidelity**: CAN frame structure, timing behavior, and response sequences matched the physical hardware precisely enough to run the full test suite without modification
- **Qt framework**: Used Qt's event loop and signal/slot mechanism to manage concurrent CAN channel simulation
- **CI integration**: With no physical hardware dependency, the simulator ran in standard Linux CI environments, enabling overnight parallel test execution across multiple agents

## Results

- Hardware tool cost per developer seat: USD 5K-14K → $0
- 1,000+ test cases parallelized across CI agents and run overnight unattended
- Development teams no longer blocked on hardware availability for test execution
- Test coverage increased as running tests became low-friction

## Tech Stack

| Component | Technology |
|---|---|
| Language | C++ |
| Framework | Qt |
| Protocol | CAN Bus |
| Domain | Automotive Head Unit, vehicle communication |
