# CloneTrace

### Detect the clone. Survive the disguise. Explain the evidence.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-Backend-green)
![React](https://img.shields.io/badge/React-Frontend-61DAFB)
![License](https://img.shields.io/badge/License-All%20Rights%20Reserved-red)
![Hackathon](https://img.shields.io/badge/Hack2Innovate-PS--08%20Mobile%20Security-orange)

CloneTrace is an **explainable, obfuscation-resilient Android clone and impersonation forensics system** that compares a baseline APK against a candidate APK using multiple independent evidence signals.

> **Most detectors tell you that two APKs are similar.**
> **CloneTrace shows you why they're similar, what changed, and whether the clone became dangerous.**

---

## The Problem

A fake or repackaged Android application can be far more sophisticated than a simple copy. It may:

- **Reuse** the original application's code structure and business logic
- **Preserve** the original resources, assets, and layouts
- **Imitate** branding through icon similarity and string preservation
- **Change** the package namespace to avoid automated detection
- **Change** the signing certificate to bypass integrity checks
- **Rename** classes and methods to defeat shallow code comparison
- **Alter** strings and UI text just enough to claim uniqueness
- **Add** dangerous permissions (SMS, Accessibility, Contacts)
- **Add** malicious network endpoints to exfiltrate data
- **Preserve** enough structure to remain recognizably derived from the original

A single similarity percentage answers none of the critical forensic questions:

- *How similar, and in which evidence dimensions?*
- *Which evidence survived transformation?*
- *What was added, removed, or modified?*
- *Is the candidate merely repackaged, or was it weaponized?*
- *Where do different signals disagree — and why?*

---

## The CloneTrace Approach

CloneTrace performs multi-dimensional static APK forensics and produces an **explainable comparison**, not only a score.

```
Baseline APK + Candidate APK
            │
            ▼
       APK Ingestion
            │
    ┌───────┼────────┐
    ▼       ▼        ▼
 Identity  Content   Code
    │       │        │
    └───────┼────────┘
            ▼
      Evidence Fusion
            │
     ┌──────┼──────┐
     ▼      ▼      ▼
   Clone  Brand  Threat
   Conf.  Conf.  Conf.
            │
            ▼
        Delta Engine
     (Preserved / Added
      Modified / Removed)
            │
            ▼
      Explainable Verdict
      + Smoking Gun
      + Signal Disagreement
      + Clone DNA
            │
       ┌────┼────┐
       ▼    ▼    ▼
  Dashboard JSON  Judge Mode
```

---

## Full Architecture

```
                    CLONETRACE
                         │
                         ▼
                  APK INGESTION
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       IDENTITY       CONTENT          CODE
          │              │              │
       Cert           Icon pHash      DEX
       Package        Strings         API
       Manifest       Resources       Classes
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  EVIDENCE ENGINE
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
         CLONE         BRAND        THREAT
        CONFIDENCE    CONFIDENCE   CONFIDENCE
            │            │            │
            └────────────┼────────────┘
                         ▼
                   WHAT CHANGED?
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      PRESERVED        ADDED         MODIFIED
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                    SMOKING GUN
                         │
                SIGNAL DISAGREEMENT
                         │
                     CLONE DNA
                         │
                         ▼
                   FINAL VERDICT
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      DASHBOARD       JSON/REPORT    JUDGE MODE
```

---

## Multi-Dimensional APK Evidence

CloneTrace extracts and fuses evidence from three layers:

### Identity & Provenance

| Field | Description |
|:------|:------------|
| APK SHA-256 | Exact binary identity. Identical SHA-256 = exact binary match. Different SHA-256 does not mean unrelated. |
| Package Name | Application namespace — renaming is a common disguise technique |
| App Label | Displayed application name |
| Version Name / Code | Declared version identity |
| Min / Target SDK | SDK configuration |
| Certificate SHA-256 / SHA-1 | Signing certificate fingerprint |
| Certificate Issuer / Subject / Serial | Certificate metadata |
| Validity Period | Certificate validity window |

> **Important**: A certificate mismatch means *different signing provenance*. It is not automatically evidence of malicious intent. Self-signed certificates are common in legitimate independent releases.

### Visual / Icon Evidence

CloneTrace extracts the application icon and computes a **perceptual hash (pHash)** to measure visual similarity independent of file-level differences.

> **Limitation**: Android adaptive icons defined as XML vector drawables may not produce a hashable rasterized image. When visual evidence is unavailable, it is marked as `UNAVAILABLE` and treated as **missing evidence**, not as dissimilarity. The evidence math remains intact.

### Code / DEX Evidence

CloneTrace parses DEX bytecode via Androguard and extracts:

- **Application-specific class names** — filtered to the application's own package namespace
- **Method and field structure** — structural fingerprint of the application's code organization
- **API call sets** — which Android APIs and external endpoints are referenced
- **Class DNA** — a hierarchical structural fingerprint comparing what classes, methods, and API references survived transformation

> **Why not raw file hashes?** A simple DEX byte-level hash breaks after any recompilation, string change, or method addition — even trivial ones. CloneTrace is designed to extract structural relationships that survive common disguise transformations.

### Manifest & Component Evidence

- Declared permissions (uses-permission)
- Exported components (Activities, Services, Receivers, Providers)
- Intent filters and action declarations
- Application metadata

### Network Evidence

- HTTP/HTTPS endpoint extraction
- Suspicious domain and IP detection

### Native Libraries

- Presence and naming of `.so` native libraries

---

## Clone DNA

Clone DNA is the multi-dimensional structural fingerprint of the APK comparison. Each dimension captures a different aspect of similarity:

| Dimension | What It Measures |
|:----------|:----------------|
| `identity` | Package name and signing certificate alignment |
| `visual` | Perceptual icon similarity (where available) |
| `resources` | Resource file structure overlap |
| `code` | Application-specific class/method structure |
| `api` | Android API reference overlap |
| `network` | Network endpoint overlap |
| `native` | Native library presence overlap |

Clone DNA answers: **"What parts of the application survived the disguise?"**

Each dimension reports a score and an `availability` flag. Unavailable dimensions do not penalize the score — they are excluded from the evidence fusion calculation.

> **Known Limitation**: Bundled third-party SDK/framework overlap (e.g., Firebase, Kotlin Coroutines, Google Play Services) can contribute structural similarity between unrelated applications. Application-focused evidence filtering reduces but does not eliminate this pressure.

---

## Explainable Evidence Fusion

CloneTrace combines multiple independent signals rather than relying on a single similarity number. The underlying design principle is:

> **Different transformations affect different evidence dimensions.**

| Transformation | Primary Evidence Affected |
|:---------------|:--------------------------|
| Package rename | Identity dimension |
| Certificate change | Provenance / identity |
| Class/method rename | Code names (but structural DNA can survive) |
| String modification | Resource content evidence |
| Icon modification | Visual / perceptual evidence |
| Permission addition | Security delta / threat evidence |
| Malicious endpoint addition | Network / security delta |

Unavailable evidence is handled as **missing**, not as dissimilarity. The scoring formula adapts to available evidence, preserving mathematical integrity across different APK types.

---

## Three Different Questions, Three Different Scores

### Clone Confidence
*How strongly does the candidate appear structurally derived from the baseline?*

Measures structural, code, resource, and identity overlap.

### Brand Confidence
*How strongly does the candidate preserve recognizable branding and content signals?*

Measures visual, string, and content similarity — elements a user would recognize.

### Threat Confidence
*How strongly do observed changes indicate suspicious or security-relevant behavior?*

Measures security-sensitive capability additions, dangerous permission deltas, suspicious network endpoints, and malicious component additions.

**This separation is a core design principle.** Consider:

| Scenario | Clone | Brand | Threat |
|:---------|:-----:|:-----:|:------:|
| Exact binary copy | High | High | Low |
| Legitimate re-release / re-sign | High | High | Low |
| Repackaged (renamed) | High | Low/Mid | Low |
| **Trojanized clone** | **High** | **Variable** | **High** |
| Unrelated app with shared SDK | Mid | Low | Low |

---

## What Changed?

The Delta Engine identifies forensic differences between baseline and candidate across four categories:

- **PRESERVED** — Evidence present and identical in both APKs
- **ADDED** — Evidence present only in the candidate (new capabilities, endpoints, permissions)
- **MODIFIED** — Evidence that changed between baseline and candidate
- **REMOVED** — Evidence present only in the baseline

Delta is analyzed across: Manifest (permissions, components), Network (endpoints), Resources, Capabilities, and API references.

The delta answers: **"What was the attacker trying to add or hide?"**

---

## Smoking Gun

The dashboard highlights the single strongest piece of evidence supporting the verdict — the **Smoking Gun**.

This gives a reviewer an immediate focus point for investigation. It is derived from the highest-confidence evidence item across all dimensions.

> The Smoking Gun is an evidence-prioritization heuristic. It should be read as "the most notable evidence," not as a definitive proof.

---

## Signal Disagreement

Different evidence dimensions can disagree. CloneTrace surfaces these contradictions explicitly.

**Examples of meaningful disagreements:**
- High code structural similarity + changed package identity → possibly disguised repackage
- High structural clone evidence + weak visual evidence → icon was changed to impersonate differently
- High clone confidence + high threat confidence → candidate preserved structure while adding dangerous capabilities

Signal disagreement itself is **useful forensic information** — it characterizes the transformation strategy used by the clone author.

---

## Security Delta

CloneTrace performs security-focused delta analysis to identify:

- Newly added dangerous/sensitive permissions (e.g., `SEND_SMS`, `READ_CONTACTS`)
- Accessibility service declarations
- New exported components with broad intent filters
- New network endpoints (especially HTTP or suspicious domains)
- Security-sensitive capability additions

> CloneTrace does not automatically classify every certificate change or package rename as malicious. Threat Confidence is specifically driven by security-relevant evidence deltas, not by identity changes alone.

---

## Trojanized Clone Detection

A **Trojanized Clone** is an application that preserves enough original structure to appear derived from a known legitimate app, while introducing security-relevant additions.

CloneTrace identifies this pattern through the combination of:

```
High Clone Confidence
       +
High Threat Confidence
       =
Potential TROJANIZED CLONE
```

**Benchmark demonstration (B10):**

| Score | Value | Meaning |
|:------|------:|:--------|
| Clone Confidence | 80 | Strong structural derivation from original |
| Brand Confidence | 0 | Identity changed (package renamed) |
| Threat Confidence | 55 | Significant security-relevant additions detected |
| **Verdict** | **TROJANIZED CLONE** | Combined evidence indicates weaponized derivative |

> This result is from our controlled adversarial benchmark. It demonstrates the evidence logic — it is not a claim of real-world detection accuracy.

---

## Obfuscation-Resilient Design

CloneTrace is **designed to remain useful under common disguise and repackaging transformations**. Common transformations affect different evidence layers:

- Package renaming changes namespace evidence but not code structure
- Certificate changes affect identity evidence but not application logic
- Class/method renaming changes symbol names but not structural organization
- String changes affect content evidence but not architectural patterns

By operating across multiple independent dimensions simultaneously, CloneTrace maintains useful evidence even when individual dimensions are fully transformed.

> **CloneTrace does not claim immunity to obfuscation.** A determined adversary who rewrites application logic, replaces all resources, and changes network endpoints will reduce evidence across all dimensions. The system is designed to be resilient against common repackaging — not against complete rewrites.

---

## Controlled Adversarial Benchmark

CloneTrace is evaluated against a reproducible adversarial benchmark of 12 controlled variants (B0–B11) derived from a single baseline APK.

| Variant | Transformation | Clone | Brand | Threat | Verdict |
|:--------|:--------------|------:|------:|-------:|:--------|
| **B0** | Original (baseline = candidate) | 100 | 100 | 0 | EXACT BINARY MATCH |
| **B1** | Package rename only | 80 | 0 | 0 | INSUFFICIENT EVIDENCE |
| **B2** | Re-sign only | 94 | 100 | 0 | REPACKAGED APP |
| **B3** | Package rename + re-sign | 80 | 0 | 0 | INSUFFICIENT EVIDENCE |
| **B4** | Class/method rename | 85 | 100 | 0 | REPACKAGED APP |
| **B5** | String modification | 85 | 100 | 0 | REPACKAGED APP |
| **B6** | Icon modification | 85 | 100 | 0 | REPACKAGED APP |
| **B7** | Add SEND\_SMS permission | 85 | 100 | 15 | REPACKAGED APP |
| **B8** | Add Accessibility service | 85 | 100 | 15 | REPACKAGED APP |
| **B9** | Add malicious HTTP endpoint | 85 | 100 | 10 | REPACKAGED APP |
| **B10** | Package rename + malicious modifications | 80 | 0 | 55 | **TROJANIZED CLONE** |
| **B11** | Unrelated flashlight APK (hard negative) | 57 | 0 | 10 | SUSPICIOUS DERIVATIVE |

> **These are controlled benchmark results, not real-world accuracy metrics.** Precision, recall, and false-positive rates on a large external dataset have not been measured.

### B10 vs B11: The Critical Separation

| | B10 (Trojanized Clone) | B11 (Unrelated App) |
|:|:----------------------:|:-------------------:|
| Clone Confidence | **80** | **57** |
| Threat Confidence | **55** | **10** |
| Verdict | TROJANIZED CLONE | SUSPICIOUS DERIVATIVE |
| Separation | **23 Clone pts / 45 Threat pts** | |

The B10/B11 separation on our controlled benchmark demonstrates that the evidence fusion correctly distinguishes a trojanized clone from an unrelated application.

**B11 is an intentional known-limitation case.** The unrelated flashlight APK scores 57% Clone Confidence due to structural overlap from shared third-party SDKs (e.g., Firebase, Kotlin runtime). This is acknowledged as **false-positive pressure** inherent in static structural comparison and is a known limitation of the current system.

---

## Interactive Forensic Console

The CloneTrace frontend is a **premium dark cyber-forensics dashboard** built for both technical review and judge-friendly presentation.

| Feature | Description |
|:--------|:------------|
| **APK Upload** | Drag-and-drop or file-select for Baseline and Candidate APKs |
| **Identity Panel** | Side-by-side package name, version, certificate SHA-256 comparison |
| **Verdict Hero** | Large, immediate verdict with color-coded severity |
| **Score Rings** | Animated circular gauges for Clone / Brand / Threat confidence |
| **Smoking Gun** | Highlighted strongest evidence supporting the verdict |
| **Security Delta** | Enumerated high-risk additions detected in the candidate |
| **Clone DNA Grid** | Per-dimension breakdown of 7 evidence categories with availability flags |
| **What Changed?** | Tabbed view: Preserved / Added / Modified / Removed |
| **Evidence Breakdown** | Math-transparent accordion showing per-signal contribution to scores |
| **Signal Disagreement** | Card highlighting contradictions between evidence dimensions |
| **Benchmark Page** | Interactive B0–B11 controlled experiment runner |
| **JSON Export** | Full deterministic forensic payload export |
| **Judge Mode** | Full-screen, distraction-free forensic summary for live demonstration |

---

## Technical Stack

### Backend

| Component | Technology |
|:----------|:-----------|
| API Framework | FastAPI (Python) |
| APK Parsing | Androguard |
| APK Decompilation | Apktool (external dependency) |
| Image Analysis | Pillow + ImageHash (pHash) |
| String Similarity | rapidfuzz |
| Data Modeling | Pydantic v2 |
| Testing | pytest |

### Frontend

| Component | Technology |
|:----------|:-----------|
| Framework | React 18 |
| Build Tool | Vite |
| Styling | Tailwind CSS |
| Icons | Lucide React |
| Utilities | clsx + tailwind-merge |

---

## Repository Structure

```
CloneTrace/
├── backend/
│   ├── app/
│   │   ├── analyzers/       # Evidence extractors
│   │   │   ├── identity.py  #   Certificate, package, APK metadata
│   │   │   ├── visual.py    #   Icon pHash comparison
│   │   │   ├── structural.py#   DEX class/method/API analysis
│   │   │   ├── manifest.py  #   Permission and component extraction
│   │   │   ├── network.py   #   Endpoint extraction
│   │   │   ├── native.py    #   Native library analysis
│   │   │   └── api.py       #   API reference analysis
│   │   ├── engine/          # Evidence fusion
│   │   │   ├── comparator.py#   Evidence comparison logic
│   │   │   ├── scorer.py    #   Clone / Brand / Threat confidence
│   │   │   ├── intelligence.py# Clone DNA, Signal Disagreement
│   │   │   ├── security.py  #   Security delta engine
│   │   │   └── benchmark.py #   Benchmark runner
│   │   ├── models/          # Pydantic data models
│   │   │   ├── evidence.py  #   Evidence data structures
│   │   │   └── delta.py     #   Delta model
│   │   └── main.py          # FastAPI routes and application
│   ├── tests/
│   │   ├── test_api.py      # API endpoint tests
│   │   └── test_models.py   # Data model tests
│   ├── requirements.txt
│   └── run.py               # Backend entry point
├── benchmark/
│   ├── original/            # B0 baseline APK
│   ├── variants/            # B1–B11 generated variant APKs
│   └── scripts/             # Automated variant generation (generate.py)
├── frontend/
│   ├── src/
│   │   ├── App.jsx          # Application shell, routing, upload interface
│   │   ├── Dashboard.jsx    # Forensic results dashboard
│   │   └── index.css        # Cyber-forensics design system
│   ├── tailwind.config.js
│   └── vite.config.js
├── docs/                    # Technical specification documents
└── README.md
```

---

## Local Setup & Running

### Prerequisites

- Python 3.10+
- Node.js 18+
- [Apktool](https://apktool.org/) — required only for benchmark variant generation, not for running the application

### Backend

```powershell
# From the project root:
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r backend/requirements.txt

# Start the backend:
python backend/run.py
```

- Backend: **http://localhost:8000**
- Health check: **http://localhost:8000/api/v1/health**

### Frontend

```powershell
cd frontend
npm install
npm run dev
```

- Frontend: **http://localhost:5173**

### Running Tests

```powershell
# From project root, with venv active:
$env:PYTHONPATH = "backend"
pytest
```

```powershell
# Frontend production build check:
cd frontend
npm run build
```

---

## 3-Minute Demo Flow

1. **Open CloneTrace** at `http://localhost:5173`
2. **Upload** `benchmark/original/B0.apk` as the Baseline
3. **Upload** `benchmark/variants/B10.apk` as the Candidate
4. Click **Analyze Evidence**
5. Observe the result: **Clone 80 / Brand 0 / Threat 55 → TROJANIZED CLONE**
6. Review the **Smoking Gun** — the strongest supporting evidence
7. Open **What Changed** → Added tab — see security-relevant additions
8. Review the **Security Delta** — high-risk capability additions
9. Examine the **Clone DNA grid** — which dimensions survived transformation
10. Open **Evidence Breakdown** — see the exact math behind every score
11. Run **B11** (unrelated flashlight APK) to demonstrate the hard-negative case: Clone 57, Threat 10 → `SUSPICIOUS DERIVATIVE`
12. Enter **Judge Mode** for the clean full-screen forensic summary

> *"Most detectors tell you that two APKs are similar. CloneTrace shows you why they're similar, what changed, and whether the clone became dangerous."*

---

## Why CloneTrace?

| Approach | Traditional Similarity | CloneTrace |
|:---------|:-----------------------|:-----------|
| Output | Single similarity score | Multi-dimensional evidence + verdict |
| Explainability | Opaque percentage | Per-signal contribution breakdown |
| Evidence | One signal | 7+ independent evidence dimensions |
| Threat detection | Not separated | Dedicated Threat Confidence score |
| Delta analysis | None | Preserved / Added / Modified / Removed |
| Missing evidence | Not addressed | Flagged as unavailable, excluded from math |
| Presentation | Raw number | Judge Mode, Smoking Gun, Clone DNA, JSON export |
| Benchmark | None | 12-variant controlled adversarial benchmark |

CloneTrace's design focuses on:
- **Evidence-centered similarity fusion** across independent dimensions
- **Application-focused structural evidence** that reduces generic SDK noise
- **Separate Clone / Brand / Threat confidence** dimensions
- **Malicious-delta analysis** distinguishing repackaged from trojanized
- **Explainable verdicts** with a transparent evidence math trail
- **Controlled adversarial benchmark** methodology
- **Judge Mode** forensic comparison workflow

---

## Current Limitations

- **Static analysis only** — CloneTrace does not execute APKs in a dynamic sandbox. Dynamic behavior (runtime code loading, network calls during execution) is not analyzed.
- **Adaptive XML icon limitation** — Android adaptive icons defined as vector XML drawables may not produce a hashable rasterized image. Visual evidence is marked `UNAVAILABLE` in these cases and excluded from scoring.
- **Third-party SDK overlap** — Bundled frameworks (Firebase, Kotlin Coroutines, Google Play Services) contribute structural similarity between unrelated applications. B11 at 57% Clone Confidence demonstrates this pressure. Application-focused class filtering reduces but does not eliminate it.
- **String/resource analysis** — Current string and resource analysis uses structural hashing rather than deep semantic comparison. Sophisticated obfuscation of string content may reduce evidence quality.
- **Controlled benchmark scope** — The B0–B11 benchmark covers common transformation patterns but is not a substitute for statistical evaluation on a large external dataset. No precision/recall figures are claimed.
- **No obfuscation immunity** — A complete application rewrite will reduce evidence across all dimensions.
- **No automated takedown** — CloneTrace produces forensic analysis for human review. It does not automatically submit DMCA or app-store takedown requests.

---

## Responsible Use

CloneTrace is intended for:

- **Authorized APK analysis** — analysis of APKs you own or have authorization to analyze
- **Defensive security research** — investigating potential clone/impersonation attacks against your own applications
- **App-store and platform moderation workflows** — supporting human review of suspected clones
- **Controlled benchmarking and security research** — evaluating clone detection methodology

Do not use CloneTrace to analyze applications without authorization. Forensic output should be reviewed by a qualified human analyst before any enforcement action.

---

## Outputs

Each analysis produces:

- **Interactive dashboard** — full forensic comparison with all evidence dimensions
- **Verdict** — one of: `EXACT BINARY MATCH`, `REPACKAGED APP`, `SUSPICIOUS DERIVATIVE`, `TROJANIZED CLONE`, `INSUFFICIENT EVIDENCE`
- **Three confidence scores** — Clone / Brand / Threat (0–100)
- **Evidence breakdown** — per-signal contribution to each score
- **Delta report** — Preserved / Added / Modified / Removed per evidence category
- **Clone DNA** — per-dimension availability and score
- **Security delta** — high-risk additions list
- **Smoking Gun** — strongest supporting evidence
- **Signal disagreements** — logical contradictions between evidence dimensions
- **Deterministic JSON export** — full forensic payload for downstream processing

> Forensic output represents a static analysis report. It is not a legal certification or guarantee of ground truth.

---

## Project Status

**Hack2Innovate CTF + Hackathon 2026 — PS-08 Mobile Security**

---

## Repository

**https://github.com/Krishn0x/CloneTrace**
