# Executive Summary — A17 Pro Shared I²C4 Fault (Secure Enclave + Digitizer)

## What was found

A reproducible **silicon-level design defect** in Apple’s **A17 Pro (D84AP)** places the **Secure Enclave Processor (SPU)** and the **digitizer controller** on the **same I²C4 bus**. Under **bus instability**—**commonly** from **power degradation** (e.g., brown-outs), **battery reseat/connector bounce**, or other I²C noise/EMI sources—**both subsystems fail at early boot**: the SPU stalls in **SecureROM** and the digitizer mis-enumerates (invalid transducers). Security guarantees (trusted boot, biometrics, keychain) and basic usability (touch input) are simultaneously lost until an eventual recovery cycle. Because **rose** aggressively prunes early-boot logs, **forensic artifacts are largely erased** before standard triage tools can capture them. The defect is **architectural** and **not remediable in software**.

**Scope & status**

* **Hardware/SoC:** iPhone 15 Pro Max (A17 Pro, **D84AP**, TSMC 3 nm)
* **Firmware context:** Observed at early boot; **iBoot 11881.80.57** documented
* **Repro trigger:** Induced **I²C4 instability** (e.g., brown-out / battery reseat) during bring-up

**Representative indicators (serial/console)**

```
aoprose: ... FWState::SecureROM
AppleSPU::_handleReadyReport, serviceName (arc-eeprom-i2c)
Couldn't alloc class "AppleSPULogDriver"
IOHIDEventDriver: Invalid digitizer transducer
```

---

## Root cause vs. proximate triggers

* **Root cause (architectural):** **SPU and digitizer share I²C4**, creating a single **fault domain** without isolation between a **trusted security** subsystem and a **UI-critical** peripheral.
* **Common proximate triggers (non-exhaustive):**

  * **Power degradation:** brown-outs, battery reseat, connector bounce/partial contact
  * **Signal integrity/noise:** I²C4 EMI, marginal pull-ups, trace impedance/tolerance issues
  * **Contention/glitches:** transient bus contention during bring-up or peripheral reset

---

## Why it matters (executive view)

| Dimension        | Outcome (plain language)                                                                           |
| ---------------- | -------------------------------------------------------------------------------------------------- |
| **Security**     | SPU never reaches a trusted state → **no secure boot attestation / no biometrics / no keychain**   |
| **Availability** | Digitizer fails at bring-up → device appears **bricked** or intermittently unstable                |
| **Forensics**    | **Rose** prunes early-boot evidence → post-recovery captures look clean (**blind spot**)           |
| **Tamper model** | Physical-adjacent actor can **induce I²C faults** to mask hands-on tampering                       |
| **Compliance**   | Attestation gaps jeopardize **MDM posture**, regulated workflows, and **chain-of-custody**         |
| **Business**     | Increased “no-fault-found” returns, support churn, and reputational risk in high-assurance markets |

**Severity**

* **Critical (HW-ARCH / Unpatchable)** — simultaneous impact on **security**, **availability**, and **forensics**, with **tamper-masking** potential.

---

## Immediate actions (7–14 days)

1. **Flag & quarantine** symptomatic units; annotate RMA/FA workflows with this signature.
2. **Avoid intentional power cycling** of mission-critical devices in custody; treat brown-outs as **trust-interrupting events**.
3. **Mandate early-boot capture**: require serial console collection to preserve SPU/digitizer lines **before recovery**.
4. **Advise high-risk customers** (defense/finance/LE) on possible **attestation gaps** and **evidence loss**; update handling SOPs.
5. **Open focused FA track** on **I²C4 signal integrity** (impedance, pull-ups, routing/noise tolerance, isolation).


## Engineering recommendations (roadmap)

* **Hard isolation:** Place **SPU** and **UI-critical peripherals** on **separate physical buses**; remove single points of failure.
* **Redundancy & monitoring:** Secondary **EEPROM/SPU access paths**, **I²C health monitors**, and **graceful degradation** paths.
* **BootROM breadcrumbs:** **Non-volatile, tamper-evident fault logging** for pre-OS events (SPU ready-state, I²C faults).
* **Early attestations:** Expose structured, rate-limited **SPU status** early enough for custody tools to capture.

---

### One-paragraph elevator pitch

A single shared I²C4 line in the A17 Pro lets routine power disturbances take down both the Secure Enclave and the touchscreen at boot, collapsing the device’s root of trust while leaving almost no evidence. It’s reproducible on retail hardware, **unfixable in software**, and uniquely dangerous in high-assurance settings because **rose** prunes the only telltale logs. Isolate symptomatic units now, capture early-boot serial logs, warn regulated customers about trust interruptions, and prioritize a silicon redesign that separates fault domains, adds redundancy, and preserves tamper-evident breadcrumbs.

---


-- Joseph Goydish II
