---
title: S-Band Telemetry Waveform Product
description: Led product design for an S-Band telemetry waveform - LDPC encoder/decoder on FPGA and complete MAC/HAL embedded software stack. Secured recurring aerospace defense contracts.
---

# S-Band Telemetry Waveform Product

!!! abstract "Summary"
    **Type**: Product development - aerospace/defense
    **Stack**: C, FPGA, RTOS

    **Outcomes:**

    - LDPC encoder/decoder implemented on FPGA from M.Sc. research baseline
    - Complete MAC layer and HAL software stack developed
    - Product secured recurring aerospace defense contracts

## Challenge

Develop a production S-Band telemetry waveform product for aerospace/defense use. The waveform required a high-performance LDPC (Low Density Parity Check) forward error correction layer to meet link budget requirements in challenging RF environments.

LDPC codes are capacity-approaching - they perform close to the Shannon limit - but their hardware implementation is non-trivial: the iterative belief-propagation decoding algorithm requires careful architecture to achieve the necessary throughput on FPGA with acceptable resource utilization.

## Approach

Led product design across two layers:

**FPGA - LDPC encoder/decoder**

Implemented the LDPC encoder and belief-propagation decoder directly in FPGA logic. The implementation drew on M.Sc. research (see below) in hardware channel coding, adapting the academic decoder architecture for production constraints: fixed-point arithmetic, pipelined throughput, and resource budgeting for the target device.

**Embedded software - MAC/HAL stack**

Developed the complete MAC (Medium Access Control) layer and HAL (Hardware Abstraction Layer) software stack in C on an RTOS. The software stack managed framing, timing, synchronization, and the interface between the application layer and the FPGA waveform core.

## Results

- Complete S-Band telemetry waveform product delivered
- Recurring aerospace defense contracts secured
- LDPC implementation from research → production, demonstrating the direct path from M.Sc. work to shipped hardware

## Tech Stack

| Layer | Technology |
|---|---|
| FPGA | LDPC encoder/decoder (belief propagation) |
| Embedded | C on RTOS |
| Software | MAC layer, HAL |
| Standard | S-Band telemetry waveform |

---

## Research { #research }

**M.Sc. Thesis: Hardware Implementation of Optimum Channel Codes**

School of Electrical and Electronic Engineering, Nanyang Technological University, Singapore - 2011

Investigated Low Density Parity Check (LDPC) codes: capacity-approaching block codes whose iterative belief-propagation decoding algorithm outperforms most existing forward error correction schemes. The thesis covered code construction, hardware decoder architecture for FPGA implementation, and performance characterization under AWGN channel conditions.

The research formed the direct academic foundation for the FPGA-based LDPC implementation in the S-Band telemetry product above.

`LDPC` `Channel Coding` `FPGA` `Signal Processing` `Hardware Implementation`

[:material-link: View thesis (NTU Digital Repository)](https://hdl.handle.net/10356/53479){ .md-button target="_blank" }
