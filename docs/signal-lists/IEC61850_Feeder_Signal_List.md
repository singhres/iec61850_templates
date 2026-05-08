# IEC 61850 Signal List — Typical 11 kV Distribution Feeder Bay

**Scope:** IED-agnostic signal definitions for a standard 11 kV distribution feeder bay.  
These definitions are intended to guide SCD engineering, dataset/RCB configuration, and SCADA mapping.

---

## Contents

1. [Conventions](#conventions)
2. [Signal Groups](#signal-groups)
   - [Switchgear Status](#1-switchgear-status)
   - [Pickups & Sequence of Events (SOE)](#2-pickups--sequence-of-events-soe)
   - [Protection Trips (GOOSE)](#3-protection-trips-goose)
   - [IED & Communication Health](#4-ied--communication-health)
   - [Asset Condition](#5-asset-condition)
   - [Measurements](#6-measurements)
   - [Control](#7-control)
3. [Dataset Summary](#dataset-summary)
4. [RCB / GCB Parameter Defaults](#rcb--gcb-parameter-defaults)
5. [Engineering Notes](#engineering-notes)

---

## Conventions

| Abbreviation | Meaning |
|---|---|
| CDC | Common Data Class (IEC 61850-7-3) |
| LN | Logical Node class |
| DO | Data Object name |
| DA | Data Attribute(s) included in dataset |
| BRCB | Buffered Report Control Block |
| URCB | Unbuffered Report Control Block |
| GCB | GOOSE Control Block |
| TrgOps | Trigger options: `dchg` (data change), `qchg` (quality change), `dupd` (data update), `gi` (general interrogation), `period` (integrity period) |
| BufTm | Buffer time in milliseconds |
| IntgPd | Integrity period in milliseconds (86 400 000 = 24 h) |
| `—` | Not applicable for this transport type |

> **Note on DA notation:** `mag.f` refers to the `mag` structure's `f` (float) component within an `MV` CDC.  
> `actVal` is the accumulated value DA within an `BCR` or similar CDC used for wear counters.

---

## Signal Groups

### 1. Switchgear Status

**Dataset:** `Switchgear_Status`  
**Transport:** MMS  
**RCB type:** BRCB  
**Report name:** `RP_SWSTAT_BR`  
**TrgOps:** `dchg + qchg + dupd + gi + period`  
**BufTm:** 100 ms  
**IntgPd:** 86 400 000 ms (24 h)

| Signal | CDC | LN | DO | DA | Notes |
|---|---|---|---|---|---|
| CB Status (Open / Close) | DPS | XCBR | Pos | stVal, q, t | Primary breaker position. DPS covers both Open and Close states. |
| Isolator (Open / Closed) | SPS | XSWI | Pos | stVal, q, t | Busbar isolator. |
| Cable Earth Position | SPS | XSWI | Pos | stVal, q, t | Earth switch position. |
| Auto-reclose On | SPS | RREC | mod | stVal, q, t | AR enabled/disabled mode indication. |
| SEF Off | SPS | PTEF | mod | stVal, q, t | Sensitive earth fault function disabled. |
| Control on Local | SPS | CSWI | Loc | stVal, q, t | Local/remote control mode. **Mandatory for SCADA awareness.** |

---

### 2. Pickups & Sequence of Events (SOE)

**Dataset:** `Pickups_&_SOE`  
**Transport:** MMS  
**RCB type:** BRCB  
**Report name:** `RP_SOE_BR`  
**TrgOps:** `dchg + qchg + dupd + gi + period`  
**BufTm:** 100 ms  
**IntgPd:** 86 400 000 ms (24 h)

| Signal | CDC | LN | DO | DA | Notes |
|---|---|---|---|---|---|
| Prot O/C Trip | SPS | PTOC | Op | stVal, q, t | SOE evidence. Distinct from pickup — see below. |
| Prot E/F Trip | SPS | PTEF | Op | stVal, q, t | |
| Prot O/C HS Trip | SPS | PTOC | Op | stVal, q, t | High-set instantaneous element. SOE evidence. |
| Prot E/F HS Trip | SPS | PTEF | Op | stVal, q, t | |
| Prot O/C Pickup | SPS | PTOC | Str | stVal, q, t | Pickup (`Str`) is operationally distinct from trip (`Op`). |
| Prot E/F Pickup | SPS | PTEF | Str | stVal, q, t | |
| CB Fail | SPS | RBRF | Op | stVal, q, t | Also published via GOOSE — see Section 3. |
| Busbar Arc Trip | SPS | PARC | Op | stVal, q, t | Also published via GOOSE — see Section 3. |
| Cable Diff Trip | SPS | PDIF | Op | stVal, q, t | |

---

### 3. Protection Trips (GOOSE)

**Dataset:** `Protection_Trips`  
**Transport:** GOOSE  
**GCB name:** `GS_TRIP`  
**TrgOps / BufTm / IntgPd:** Not applicable (GOOSE is event-driven with retransmission)

| Signal | CDC | LN | DO | DA | Notes |
|---|---|---|---|---|---|
| CB Fail | SPS | RBRF | Op | stVal, q, t | Inter-IED trip via GOOSE. |
| Busbar Arc Trip | SPS | PARC | Op | stVal, q, t | Inter-IED trip via GOOSE. |

> **Design note:** CB Fail and Busbar Arc Trip appear in **both** `Protection_Trips` (GOOSE) and `Pickups_&_SOE` (MMS BRCB). This is intentional — GOOSE handles the real-time inter-IED trip function while MMS BRCB provides timestamped SOE archiving to SCADA.

---

### 4. IED & Communication Health

**Dataset:** `IED_&_Comm_Health`  
**Transport:** MMS  
**RCB type:** BRCB  
**Report name:** `RP_HEALTH_BR`  
**TrgOps:** `dchg + qchg + dupd + gi + period`  
**BufTm:** 100 ms  
**IntgPd:** 86 400 000 ms (24 h)

| Signal | CDC | LN | DO | DA | Notes |
|---|---|---|---|---|---|
| Relay Fail | SPS | LLN0 | Health | stVal, q, t | IED functional health (self-supervision). |
| Physical IED Health | SPS | LPHD | PhyHealth | stVal, q, t | Power supply / hardware health. |
| Time Sync Lost | SPS | LTMS | TmChSt1 | stVal, q, t | Loss of GNSS/PPS sync. Critical for SOE timestamp integrity. |
| GOOSE Fail | SPS | LGOS | St | stVal, q, t | GOOSE subscription loss detection. **Mandatory.** |

---

### 5. Asset Condition

**Dataset:** `Asset_Condition`  
**Transport:** MMS  
**RCB type:** BRCB  
**Report name:** `RP_ASSET_BR`  
**TrgOps:** `dchg + qchg + dupd + gi + period`  
**BufTm:** 100 ms  
**IntgPd:** 86 400 000 ms (24 h)

| Signal | CDC | LN | DO | DA | Notes |
|---|---|---|---|---|---|
| CB Wear Alarm | SPS | XCBR | SumSwAmpsR | actVal, q, t | Threshold-based cumulative interrupted current alarm. |
| Trip Coil Fail | SPS | XCBR | TrCoil | stVal, q, t | Trip coil supervision. Maintenance-critical. |

---

### 6. Measurements

**Dataset:** `Measurements`  
**Transport:** MMS  
**RCB type:** URCB  
**Report name:** `RP_MEAS_UR`  
**TrgOps:** `dchg + qchg + dupd + gi + period`  
**BufTm:** 100 ms  
**IntgPd:** 86 400 000 ms (24 h)

> **Note:** Measurements use an **Unbuffered** RCB (URCB). Adjust deadband (`db`) and `IntgPd` to suit telemetry bandwidth constraints.

| Signal | CDC | LN | DO | DA | Notes |
|---|---|---|---|---|---|
| Phase A Current | MV | MMXU | A.phsA | mag.f, q, t | |
| Phase B Current | MV | MMXU | A.phsB | mag.f, q, t | |
| Phase C Current | MV | MMXU | A.phsC | mag.f, q, t | |

---

### 7. Control

> Controls are **not** reported in any dataset. They are initiated via MMS direct- or SBO-control services.

| Signal | CDC | LN | DO | DA | Notes |
|---|---|---|---|---|---|
| CB Control | DPC | CSWI | Pos | ctlVal | Direct control or SBO per site security policy. Not included in any dataset or RCB. |

---

## Dataset Summary

| Dataset name | Signals | RCB type | Report name | Transport |
|---|---|---|---|---|
| `Switchgear_Status` | CB position, isolators, earth switch, AR mode, SEF mode, local/remote | BRCB | `RP_SWSTAT_UR` | MMS |
| `Pickups_&_SOE` | All protection trips and pickups, CB fail, arc trip, cable diff | BRCB | `RP_SOE_BR` | MMS |
| `Protection_Trips` | CB fail, busbar arc trip | GCB | `GS_TRIP` | GOOSE |
| `IED_&_Comm_Health` | Relay health, physical health, time sync, GOOSE supervision | BRCB | `RP_HEALTH_BR` | MMS |
| `Asset_Condition` | CB wear alarm, trip coil fail | BRCB | `RP_ASSET_BR` | MMS |
| `Measurements` | Phase A/B/C currents | URCB | `RP_MEAS_UR` | MMS |

---

## RCB / GCB Parameter Defaults

### Buffered RCB (BRCB) — all status and SOE datasets

| Parameter | Value | Rationale |
|---|---|---|
| `TrgOps` | `dchg + qchg + dupd + gi + period` | Captures all state transitions, quality changes, and supports GI on reconnect |
| `BufTm` | 100 ms | Prevents report flooding on chattering inputs |
| `IntgPd` | 86 400 000 ms | 24-hour integrity poll — SCADA confirms point alive once per day |
| `RptEna` | `TRUE` | Pre-enabled in CID |
| `GI` | Triggered on SCADA reconnect | Ensures full dataset refresh after comms loss |

### Unbuffered RCB (URCB) — measurements

| Parameter | Value | Rationale |
|---|---|---|
| `TrgOps` | `dchg + qchg + dupd + gi + period` | Same trigger set as BRCB — data change, quality change, data update, GI, and integrity period |
| `IntgPd` | 86 400 000 ms | |
| `db` (deadband) | Site-specific | Typically 1–2% of full-scale for 11 kV feeder currents |

### GOOSE Control Block (GCB)

| Parameter | Value |
|---|---|
| `VLAN-ID` | 0 (untagged) — **review if site uses VLAN segmentation** |
| `APPID` | Near-minimum (e.g., `0x0001` range) — **must be unique per GCB per substation** |  There may be routed GOOSE considerations here
| Multicast MAC | IEC 61850 base range `01-0C-CD-01-00-00` — **assign unique per GCB** |
| `MinTime` | 1 ms |
| `MaxTime` | 10000 ms |

> ⚠️ **Risk note:** Default VLAN-ID=0, base multicast MAC, and near-minimum APPID values represent compounded L2/application/architecture risks. Engineering must assign unique values per GCB and enforce VLAN segmentation at the station/process bus switch.  Note that the Transpower OPENSCD toolchain allows you to easily set unique values for some of these attributes.

---

## Engineering Notes

1. **Dual transport for critical signals:** CB Fail (`RBRF/Op`) and Busbar Arc Trip (`PARC/Op`) appear in both GOOSE (`Protection_Trips`) and MMS BRCB (`Pickups_&_SOE`). This is the correct pattern — GOOSE carries the real-time trip function; MMS BRCB provides the timestamped SOE record.

2. **Pickup vs Trip distinction:** `PTOC/Str` (pickup) and `PTOC/Op` (trip) must be mapped as **separate signals** in SCADA. Conflating them destroys fault analysis capability.

3. **CB Status CDC:** The circuit breaker position uses `DPS` (Double Point Status), which natively represents Open, Closed, Intermediate, and Bad State. Do not use `SPS` for breaker position.

4. **GOOSE subscription supervision:** `LGOS/St` must be included in the `IED_&_Comm_Health` dataset and alarmed in SCADA. Silent GOOSE failure (no StNum increment,) is a known operational risk.

5. **Time synchronisation:** `LTMS/TmChSt1` must be alarmed. SOE timestamps are only forensically reliable when the IED is synchronised. Recommended: GNSS-sourced PPS via IRIG-B or PTP (IEEE 1588 profile per IEC 61850-9-3).

6. **SSD absence impact:** Where no System Specification Description (SSD) exists, the SCD signal set defined here serves as the optional source for dataset membership and RCB configuration. 

7. **Measurements and deadband:** Phase current deadbands should be tuned per feeder load profile. As a guide, a 1% deadband on a feeder rated at 400 A full-scale equates to 4 A — adjust to suit the actual CT ratio and SCADA poll rate/trigger options.

8. **CB Control model:** The `CSWI/Pos` (DPC) control object is not reported. Control model (direct vs SBO) must align with the substation cybersecurity policy.

---

*Community-maintained reference. For corrections or additions, raise a GitHub issue or submit a pull request against the SCL repository.*
