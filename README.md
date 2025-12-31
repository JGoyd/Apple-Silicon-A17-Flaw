# Project Overview: A17 Pro "I2C4 Zombie" Silicon Defect

This repository contains forensic documentation and proof-of-concept materials regarding a hardware-level vulnerability in the **Apple A17 Pro (T8130) SoC**. The defect involves a microarchitectural logic trapdoor where an **I2C4 bus failure** triggers an unauthorized platform security demotion to a "Zombie" state.

---

## Repository Structure

| File / Directory | Description |
| --- | --- |
| **A17 Pro Forensic Audit Tool.py** | Python script to audit **sysdiagnose** files for the `T8122` fallback. |
| **V1.0/** | Contains original report: `A17 Flaw.md`, `Executive Summary.md`, and first `README.md`. |
| **V2.0/** | Contains updated technical report: `Apple-Silicon-A17-Flaw`. |

---

## Technical Summary

The vulnerability stems from a shared dependency between the **Secure Enclave Processor (SEP)** and the **Digitizer Controller** on the I2C4 bus. Physical or induced failure on this bus prevents SEP initialization, forcing the system into a non-secure fallback state.

### Key Findings

* **Kernel Fallback:** The system switches from the production T8130 kernel to a **T8122 (Unified/M3-class) kernel**, which lacks strict SEP handshake requirements.
* **Memory Firewall Bypass:** The hardware **DART (IOMMU)** is reconfigured to `bypass-15`, effectively disabling memory isolation and enabling DMA-based exfiltration.
* **Cryptographic Collapse:** Data-at-rest encryption is bypassed as the system mounts the private data partition with `NoEncryption` flags due to SEP unavailability.
* **Inducibility:** This state is reachable via **Fault Injection (FI)** on the `VCC_MAIN` rail during the iBoot-to-Kernel handover window.

---

## Forensic Significance

This repository provides evidence that the "Zombie" state is not a random crash but a programmed architectural response. The existence of the `bypass-15` property in production silicon indicates a significant security oversight where availability is prioritized over data integrity.

---

## Usage

Documentation is provided in Markdown format. The **Forensic Audit Tool** located in the root directory is designed to ingest and parse a **sysdiagnose** file. It identifies the demoted state by scanning system logs and I/O Registry artifacts for `T8122` identifiers and DART bypass flags.

> **Note:** This research is intended for forensic auditors and hardware security researchers. The defects described are silicon-level and cannot be fully remediated via software updates alone.

---
