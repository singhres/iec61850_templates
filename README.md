# IEC 61850 SCD Engineering Templates

A community reference repository providing IEC 61850 signal definitions, dataset configurations, and SCL artefacts to support the engineering of high-quality Substation Configuration Description (SCD) files for digital substations.

The goal is to reduce inconsistency across IED signal engineering by providing tested, well-documented templates that any protection and control engineer can adopt or adapt.

---

## Contents

```
├── docs/
│   └── signal-lists/
│       └── IEC61850_Feeder_Signal_List.md   # Signal definitions for a typical 11 kV feeder bay
├── scl/
│   └── ssd/
│       └── substation.ssd                   # Example SSD for a typical distribution substation
└── README.md
```

---

## Current Coverage

| Bay Type | Signal List | SCL Artefact | Status |
|---|---|---|---|
| 11 kV Feeder | [IEC61850_Feeder_Signal_List.md](docs/signal-lists/IEC61850_Feeder_Signal_List.md) | — | ✅ Available |
| Substation (SSD) | — | [substation.ssd](scl/ssd/substation.ssd) | ✅ Available |
| Transformer | — | — | 🔜 Planned |
| Incomer | — | — | 🔜 Planned |
| Busbar / Coupler | — | — | 🔜 Planned |

---

## Signal List Structure

Each signal list document defines, for every signal in the bay:

- **CDC** — Common Data Class per IEC 61850-7-3
- **Logical Node (LN)** — LN class from IEC 61850-7-4
- **Data Object (DO)** and **Data Attribute (DA)** — specific attributes included in the dataset
- **Dataset membership** — which named dataset the signal belongs to
- **Transport** — MMS (BRCB/URCB) or GOOSE (GCB)
- **Report / GCB name** — report control block or GOOSE control block identifier
- **TrgOps, BufTm, IntgPd** — trigger options, buffer time, and integrity period
- **Engineering notes** — rationale, mandatory flags, and common pitfalls

Signal lists are IED-agnostic and intended to inform CID/SCD configuration regardless of manufacturer.

---

## Using the SSD File

The provided `substation.ssd` is a System Specification Description file that defines the single-line topology and functional signal requirements for a typical distribution substation, independent of any specific IED.  It contains one SCADA gateway, two feeders and one bus section.

It can be opened and validated using:

- **[OpenSCD](https://github.com/openscd/open-scd)** — open-source IEC 61850 SCL editor
- **[OpenSCD — /Transpower fork]** — extended fork with additional validation and workflow support - here the SLD shows correctly.

### Recommended workflow

```
SSD (this repo)
    │
    ▼
Import into OpenSCD / Transpower fork
    │
    ▼
Add IED CID files from each manufacturer tool
    │
    ▼
Engineer datasets, RCBs, GCBs against signal list templates
    │
    ▼
Export and validate SCD
```

The SSD establishes the substation topology and intended signal set before any IED-specific configuration is introduced, which is the correct engineering sequence per IEC 61850-6.

---

## Design Principles

**IED-agnostic first.** Templates define the *what* (signal, CDC, LN, dataset, transport parameters) without assuming any specific IED manufacturer or tool.

**Dataset integrity.** Signals are grouped by operational purpose into named datasets. This grouping drives report control block design and SCADA point mapping — it is not arbitrary.

**Dual transport where required.** Critical protection signals (e.g. CB Fail, Busbar Arc Trip) are defined for both GOOSE (real-time inter-IED function) and MMS BRCB (timestamped SOE archiving). This pattern is intentional and must be preserved.

**Health and supervision are mandatory.** GOOSE subscription supervision (`LGOS/St`) and time synchronisation status (`LTMS/TmChSt1`) are non-negotiable inclusions in any bay signal set.

**SSD-first engineering.** The SSD defines the authoritative signal intent. Where no SSD exists, the signal list documents in this repository serve as the functional equivalent for contractor guidance.

---

## Contributing

Contributions are welcome. Priority areas for future templates:

- Transformer bay signal list (ATCC, YLTC, PDIF, temperature/oil monitoring)
- Incomer bay signal list
- Busbar and bus coupler signal list
- Example CID extracts for common IED platforms

To contribute:

1. Fork the repository
2. Create a branch (`feature/transformer-signal-list`)
3. Follow the structure and conventions established in the feeder signal list
4. Submit a pull request with a clear description of what is added or changed

Please raise an issue first for significant additions so the scope can be agreed before engineering effort is invested.

---

## References

- IEC 61850-6:2009+AMD1:2018 — Configuration description language
- IEC 61850-7-3:2011 — Common data classes
- IEC 61850-7-4:2010 — Compatible logical node classes and data object classes
- IEC 61850-8-1:2011 — MMS mapping
- [OpenSCD project](https://github.com/openscd/open-scd)

---

## Licence

Content in this repository is provided for reference and educational use. See [LICENSE](LICENSE) for terms.
