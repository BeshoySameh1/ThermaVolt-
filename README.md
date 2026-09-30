
**Architecture principle:** RAG is a knowledge/evidence layer that resolves missing parameters; deterministic physics, constraints, optimization and thermal diagnosis remain outside the LLM and are independently reproducible.
# Thermal Fault Diagnosis and Propagation Analysis Toolkit (`thermavolt`)

**find what caused the heating, where it started, and how it spread.** It reads eCalc configurations, audits every component against datasheets and documented real-world use, and plots the thermal propagation curve and electric-load-versus-thermal-loss relationships to validate propagation performance with less heat.

## Table of Contents

1. Overview
2. Core Principles and Scope Limits
3. System Architecture and Data Flow
4. Architecture Overview
5. eCalc Ingestion
6. Component Knowledge and Evidence Retrieval
7. Configuration Compatibility Analysis
8. Thermal Model and Nodes
9. Fault and Stress Library
10. Scenario Generation and Simulation
11. Sensor and Noise Modeling
12. Diagnosis Engines
13. Visualization and Validation Outputs
14. Explainability and Reporting
15. Evaluation Harness and Metrics
16. Optional Real-World Bench Validation
17. Repository Structure
18. Tech Stack
19. Quick Start
20. Roadmap
21. Risks and Limitations
22. Contributing and License

## 1. Overview

Electrical and propulsion systems—whether in multirotors, fixed-wing aircraft, rovers, or industrial power setups—fail progressively. A degrading connector increases resistance; an undersized ESC runs hot; a failing motor bearing creates localized friction. Heat is generated, and it propagates through physical conduction, convection, and radiation to neighboring components until a failure occurs.

This repository provides a modular, reproducible toolkit to:

- **Ingest eCalc CSV files** to parse real powertrain configurations and operating points.
- **Retrieve and audit component knowledge** from manufacturer datasheets and real-world web evidence, detecting mismatches that cause under- or over-performance and excess heat.
- **Simulate thermal propagation** across interconnected electrical and mechanical thermal nodes.
- **Generate synthetic and realistic fault datasets** under varying environmental conditions and sensor noise profiles.
- **Diagnose root causes, origin nodes, and propagation paths** using threshold baselines, Bayesian physics-residuals, and neural multi-task models.
- **Plot thermal propagation curves and electric load versus thermal loss relationships** to evaluate and prove that an improved configuration runs cooler and more efficiently.

## 2. Core Principles and Scope Limits

**Core principles:**

- **Deterministic core, probabilistic wrapper:** Physics calculations and limit checks are strictly deterministic. Machine learning and Bayesian models handle uncertainty and anomaly classification.
- **Traceable evidence:** Every component limit and diagnostic conclusion links directly to a source tier (datasheet, benchmark, or anecdotal report).
- **Be reproducible:** every dataset and result comes from a seed and a config file.
- **Read eCalc CSV configurations** and audit every component against its datasheet and its documented real-world use, flagging mismatches that cause under- or over-performance and excess heat.
- **Produce thermal propagation curves and electric-load-versus-thermal-loss plots**, and use them to show that an improved configuration runs with less heat.

**Scope limits:**

- Designed for fault detection, root-cause isolation, and thermal behavior analysis.
- Does not simulate or attempt to reproduce battery thermal runaway on real hardware.
- Does not replace eCalc's own performance model. It audits configurations and explains their heat behaviour.
- Web evidence is advisory. It never overrides a datasheet limit, and the tool does not copy or republish third-party content beyond short cited excerpts.

## 3. System Architecture and Data Flow

```
Inputs (eCalc CSV, Datasheets, Web Sources)
       │
       ▼
Ingestion, Knowledge & Compatibility Audit
       │
       ▼
Physics Core (Electrical Losses + Thermal RC Network)
       │
       ▼
Scenario & Dataset Generator
       │
       ▼
Multi-Engine Diagnosis (Threshold, Bayesian, Neural, Hybrid)
       │
       ▼
Visualization & Validation (Curves, Loss Plots, Limits, Comparisons)

```

### Inputs (v1)

- One or more **eCalc CSV files**, each describing one or more configurations (battery, ESC, motor, propeller, airframe, operating conditions, and the results eCalc calculated). See section 5.
- **Datasheets** (PDF or extracted text) supplied by you, plus optional web retrieval of further evidence. See section 6.
- Optional **measured logs** from a bench or flight test. See section 16.

### Thermal nodes (v1)

- **N1:** LiPo Battery Pack (bulk core temperature)
- **N2:** Main Power Connectors / XT60 / QS8
- **N3:** Power Distribution Board / Wiring Harness
- **N4:** Electronic Speed Controller (ESC) Logic / BEC Circuitry
- **N5:** ESC Power Stage / MOSFETs
- **N6:** Motor Stator Windings
- **N7:** Motor Bell / Permanent Magnets
- **N8:** Propeller / Aerodynamic Load Interface
- **N9:** Local Ambient Enclosure / Fuselage Cavity

## 4. Architecture Overview

Code snippet

```
flowchart TB
    subgraph IN["0. Inputs"]
        I1["eCalc CSV files"]
        I2["Datasheets (PDF)"]
        I3["Web sources<br/>test data, community reports"]
        I4["Measured logs<br/>(optional)"]
    end

    subgraph KN["1. Ingestion, Knowledge and Audit"]
        K1["eCalc parser<br/>SystemConfig and operating points"]
        K2["Evidence retriever<br/>search, extract, reconcile"]
        K3["Component cards<br/>limits, intended use, sources"]
        K4["Compatibility analyser<br/>mismatch flags and stress factors"]
    end

    subgraph CFG["2. Configuration"]
        C1["system.yaml<br/>topology and parameters"]
        C2["faults.yaml<br/>cause library"]
        C3["missions.yaml<br/>load profiles"]
    end

    subgraph SIM["3. Physics Core"]
        S1["Electrical loss model<br/>battery, contacts, ESC, motor"]
        S2["Thermal RC network<br/>ODE solver"]
        S3["Airflow and ambient model"]
    end

    subgraph GEN["4. Scenario and Data Generation"]
        G1["Scenario sampler<br/>mission, ambient, fault, severity, location"]
        G2["Fault injector"]
        G3["Sensor model<br/>noise, lag, quantization, dropout"]
        G4["Dataset builder<br/>windowing, labels, splits"]
    end

    subgraph DX["5. Diagnosis Engines"]
        D1["A. Threshold baseline"]
        D2["B. Physics-residual<br/>Bayesian inference"]
        D3["C. Neural multi-task model"]
        D4["D. Hybrid fusion"]
    end

    subgraph VIZ["6. Visualization and Validation"]
        V1["Thermal propagation curves"]
        V2["Electric load vs thermal loss"]
        V3["Temperatures vs datasheet limits"]
        V4["Configuration comparison<br/>less-heat check"]
        V5["Parity plots vs eCalc and bench"]
    end

    subgraph OUT["7. Outputs"]
        O1["Ranked causes<br/>with probabilities"]
        O2["Origin node and<br/>propagation path"]
        O3["Severity and<br/>onset time"]
        O4["Configuration audit,<br/>report and dashboard"]
    end

    subgraph EVAL["8. Evaluation Harness"]
        E1["Metrics and calibration"]
        E2["Robustness sweeps"]
        E3["Ablations"]
    end

    I1 --> K1
    I2 --> K2
    I3 --> K2
    K2 --> K3
    K1 --> K4
    K3 --> K4
    K1 --> SIM
    K4 -- "stress factors" --> SIM
    K4 -. "priors" .-> D2
    CFG --> SIM
    CFG --> GEN
    G1 --> G2 --> SIM
    SIM --> G3 --> G4
    G4 --> D1 & D2 & D3
    SIM -. healthy reference .-> D2
    D2 --> D4
    D3 --> D4
    D1 & D4 --> OUT
    SIM --> VIZ
    K1 -- "eCalc results" --> VIZ
    K3 -- "datasheet limits" --> VIZ
    I4 -.-> VIZ
    I4 -.-> EVAL
    VIZ --> OUT
    OUT --> EVAL
    G4 --> EVAL

```

**Layer responsibilities**

| Layer                          | Responsibility                                                                               | Key design rule                                                                                                |
| ------------------------------ | -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Inputs                         | eCalc CSV files, datasheets, web evidence, optional measured logs                            | Every fact carries its source                                                                                  |
| Ingestion, knowledge and audit | Parses eCalc files, builds component cards, checks compatibility                             | Numeric verdicts come from deterministic rules on sourced values. Language models only read and summarise text |
| Configuration                  | Single source of truth for topology, parameters, faults, missions                            | No magic numbers in code                                                                                       |
| Physics core                   | Turns constraint-driven operating points into electrical losses and node temperatures       | Deterministic given parameters and seed; preserves mechanical-to-electrical causality                        |
| Scenario and data generation   | Produces labelled, noisy, sensor-realistic datasets                                          | Labels come from the injected fault, never inferred                                                            |
| Diagnosis engines              | Map observed signals to causes                                                               | All engines share one input and output contract                                                                |
| Visualization and validation   | Thermal curves, load-versus-loss plots, limit checks, configuration comparison, parity plots | Every plot states its data source and uncertainty                                                              |
| Outputs                        | Constraint driver, electrical heat source, probabilities, origin, severity, path, audit, report | Always separate mechanical cause, electrical heat source, and thermal propagation; report uncertainty |
| Evaluation harness             | Measures accuracy, calibration, robustness                                                   | Held-out scenarios only                                                                                        |

### Shared diagnosis contract

Python

```
@dataclass
class DiagnosisResult:
    timestamp: float
    scenario_id: str
    causes: list[CauseProbability]    # ranked causes with confidence scores
    origin_node: str                 # predicted root-cause node (e.g. N5)
    propagation_path: list[str]      # ordered sequence of nodes affected
    onset_time: float                # estimated time fault began
    severity: float                  # estimated severity scalar [0, 1]
    confidence: float                  # calibrated overall confidence
    config_flags: list[str]            # compatibility findings that informed the priors

```

## 5. eCalc Ingestion

**Purpose:** turn eCalc CSV files into typed, validated configurations that the rest of the tool can audit, simulate, and plot.

> **Format note.** The exact column layout of an eCalc export depends on the calculator type and version, and is **to be confirmed from a real sample file**. To avoid hard-coding assumptions, the parser is driven by a mapping file (`configs/ecalc_mapping.yaml`) that says which column or row holds which quantity and in which unit. Supporting a different export layout means editing that file, not the code. Sample exports live in `tests/fixtures/ecalc/` and act as regression tests.

### 5.1 What is extracted

| Group         | Fields (whatever the export contains)                                                                                          |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Battery       | Cell count (S), parallel count (P), capacity, rated C-rate or maximum continuous current, internal resistance, nominal voltage |
| ESC           | Continuous and burst current rating, internal resistance, model name                                                           |
| Motor         | Manufacturer and model, Kv, winding resistance, no-load current, maximum current, mass                                         |
| Propeller     | Diameter, pitch, blade count, type                                                                                             |
| Airframe      | Mass, wing area, drag or efficiency assumptions                                                                                |
| Conditions    | Altitude, ambient temperature, airspeed                                                                                        |
| eCalc results | Current, power, rpm, thrust, efficiency, flight time, and any loss or temperature fields the export provides                   |

Fields missing from the export stay **unknown**. They are never silently guessed, and the audit reports which checks could not run because of missing data.

### 5.2 Processing steps

1. **Read** each CSV. A file may hold several configurations, and each gets a stable `config_id`.
2. **Map** columns to canonical fields using `ecalc_mapping.yaml`.
3. **Normalise units** (mAh to Ah, g to N, rpm, and so on) with explicit conversion tables.
4. **Validate** with a Pydantic schema and sanity checks (for example, pack voltage consistent with cell count, efficiency between 0 and 1).
5. **Report problems** per row. Unparseable rows are listed, not dropped.
6. **Emit** `SystemConfig` and `OperatingPoint` objects (for example, hover, cruise, full throttle) as JSON under `configs/ingested/`.

### 5.3 How the results are used

- The operating points become the **load profile** that drives the physics core (section 8).
- The eCalc-reported results (current, power, efficiency, and any loss or temperature values) become the **reference values** for the parity plots in section 13. Agreement with them is a consistency check on the model, not proof that either is correct (see Risks).

## 6. Component Knowledge and Evidence Retrieval

**Purpose:** for every component in every configuration, gather what is known about it: its rated limits, its intended use, and how it behaves in practice. The compatibility analysis (section 7) then compares the configuration against this knowledge.

### 6.1 Source tiers

| Tier | Source                                                                                                          | Used for                                                        | Trust                                          |
| ---- | --------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- | ---------------------------------------------- |
| 1    | Manufacturer datasheets and spec pages (your PDFs in `knowledge/
├── rag.yaml                  # RAG configuration, retrieval policy and source tiers
datasheets/`, or fetched from the manufacturer) | Hard limits and rated values                                    | Highest                                        |
| 2    | Values already in the eCalc file                                                                                | Model inputs                                                    | High for consistency, not independent of eCalc |
| 3    | Independent test data (thrust-stand results, reviewer measurements)                                             | Measured efficiency, current, and temperature behaviour         | Medium to high                                 |
| 4    | Community reports (forums, reviews, build logs)                                                                 | Qualitative experience, such as "runs hot with this prop on 6S" | Low to medium. Always labelled anecdotal       |

### 6.2 Retrieval pipeline

Code snippet

```
flowchart LR
    A["Component name<br/>from eCalc config"] --> B["Identify and normalise<br/>alias table, fuzzy match"]
    B --> C["Search<br/>datasheets, test data, reports"]
    C --> D["Extract<br/>tables and text to schema"]
    D --> E["Reconcile<br/>conflicts kept, not averaged"]
    E --> F["Component card<br/>versioned JSON"]
    E --> G["Review queue<br/>manual overrides"]
    G --> F

```

1. **Identify and normalise** the component name (for example, vendor spelling variations) through an alias table and fuzzy matching, and confirm the match before searching.
2. **Search** with query templates per component type, and cache results with a retrieval date. The search respects `robots.txt` and site terms. Only links and **short cited excerpts** are stored, not copies of pages.
3. **Extract** values in two passes:
   - deterministic extraction from PDF tables and text (for example, with regular expressions and table parsing),
   - a schema-constrained language model as a fallback for messy text, where **every value must come with the source snippet it was read from**. A value with no matching snippet is rejected.
4. **Reconcile.** Conflicts between sources are kept and displayed, not averaged. For hard limits the manufacturer wins. Conflicts go to a review queue, and your manual corrections live in `knowledge/
├── rag.yaml                  # RAG configuration, retrieval policy and source tiers
overrides.yaml`.
5. **Store** the result as a versioned **component card**.

**Design rule:** language models are used to *read and summarise text*. They never decide whether a configuration passes or fails. All numeric compliance decisions come from deterministic rules applied to sourced values (section 7).

### 6.3 Component card (illustrative)

JSON

```
{
  "component_id": "motor:EXAMPLE-VENDOR-EXAMPLE-MODEL",
  "type": "motor",
  "aliases": ["example model", "EXAMPLE-MODEL-KV"],
  "limits": {
    "max_continuous_current_A": {"value": 60, "tier": 1, "ref": "datasheet p.2, table 1", "confidence": "high"},
    "max_cells_S":              {"value": 6,  "tier": 1, "ref": "datasheet p.1", "confidence": "high"},
    "max_winding_temp_C":       {"value": 120, "tier": 1, "ref": "datasheet p.3", "confidence": "medium"}
  },
  "intended_use": {
    "application": "mid-size fixed wing",
    "recommended_props": "11-14 in",
    "recommended_cells_S": "4-6",
    "ref": "manufacturer product page, retrieved 2026-09-30"
  },
  "reported_behaviour": [
    {
      "claim": "Runs warm with large props on 6S in low airflow",
      "tier": 4,
      "kind": "anecdotal",
      "url": "https://example.com/thread",
      "retrieved": "2026-09-30"
    }
  ],
  "needs_review": false
}

```

The values above are placeholders to show the structure. They are not real product data.

## 7. Configuration Compatibility Analysis

**Purpose:** detect configuration mismatches that cause under-performance, over-performance, or excess heat, and connect each one to the part of the thermal model it stresses.

### 7.1 Checks

All thresholds are configurable in `configs/compat_rules.yaml`. The defaults are common rules of thumb and are labelled as such, not as standards.

| #  | Check                      | Rule (utilisation = operating value ÷ rated value)                                                                                                                    | Stresses node |
| -- | -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- |
| 1  | Motor current              | Peak and continuous motor current against the motor's continuous and burst ratings                                                                                    | N8            |
| 2  | ESC headroom               | Peak current against the ESC continuous rating, leaving a configurable safety margin                                                                                  | N5            |
| 3  | Battery load               | Required C-rate (current ÷ capacity) against the pack's rated continuous C, plus estimated voltage sag                                                                | N1            |
| 4  | Voltage compatibility      | Pack cell count against the motor and ESC maximum cells. A hard pass or fail                                                                                          | N5, N8        |
| 5  | Propeller loading          | Predicted rpm and current for the prop, Kv, and voltage. Flags over-propping (current beyond motor limits) and under-propping (high rpm, low thrust, poor efficiency) | N8, N9        |
| 6  | Efficiency operating point | Operating efficiency against the motor's best-efficiency region                                                                                                       | N8            |
| 7  | Connector and wire rating  | Peak and continuous current against connector and wire ratings from a rating table                                                                                    | N2, N3        |
| 8  | Duty cycle                 | Time spent near rated current against thermal time constants from the model                                                                                           | all           |
| 9  | Cooling assumption         | Assumed airflow against the installation and the component's cooling needs                                                                                            | N5, N6, N9    |
| 10 | Application fit            | The configuration's application (size, weight, cell count, prop range) against the component's *intended use* from its card                                           | all           |

Each check returns one status per component:

| Status       | Meaning                                                                        |
| ------------ | ------------------------------------------------------------------------------ |
| **OK**       | Within limits with reasonable margin                                           |
| **Marginal** | Within limits, but thin margin or short-duration limits used for long duration |
| **Over**     | Operated beyond a sourced limit                                                |
| **Under**    | Component is oversized or mismatched so it works far from its efficient region |
| **Unknown**  | The needed limit or value is missing, so the check could not run               |

### 7.2 Report contents

For each flagged component the audit reports:

- the numbers compared (operating value, rated value, utilisation),
- the **source** of each limit (tier, document, page),
- the **predicted consequence** (for example, "ESC has 5% headroom at full throttle, so MOSFET temperature is expected to be the first limit reached"),
- **reported real-world behaviour** from tier 3 and 4 evidence, shown with citations and labelled anecdotal where appropriate.

Qualitative evidence can raise the *priority* of a flag. It never changes a numeric verdict on its own.

### 7.3 From mismatch to simulation

Each flag maps to a **stress factor** in the physics core (see the fault and stress library, section 9). For example, an ESC utilisation of 0.95 sets the ESC node to run near its limit, and a 1.15 motor utilisation scales the motor current accordingly. This lets the thermal model predict *where* a mismatched configuration will run hot before anything is built, and gives the diagnosis engines a sensible prior (section 12.6).

## 8A. Automatic Parameter Resolution and Model Retuning

A central design requirement of `thermavolt` is that the user should **not need to manually enter every thermal, electrical, mechanical, or environmental parameter**. The toolkit uses a controlled parameter-resolution pipeline that extracts, derives, estimates, or calibrates missing values from the strongest available evidence.

The resolution order is:

1. **User-provided measured value**
2. **User-provided technical document / datasheet**
3. **Exact value extracted from an eCalc export**
4. **Value derived from other user-provided constraints or configuration data**
5. **Value inferred from multiple related values in the same project**
6. **Manufacturer documentation discovered by the evidence-search agent**
7. **Independent technical literature / engineering references**
8. **Empirical engineering correlation or physically constrained estimate**
9. **Model-calibrated value from available eCalc/bench observations**
10. **Bounded default only as a last resort**

The system must never silently convert an estimate into a measured or manufacturer-specified fact. Every resolved parameter is stored with:

```yaml
parameter:
  name: motor_winding_resistance
  value: 0.011
  unit: ohm
  status: derived
  source_type: user_constraint
  source_ref: project_constraint_analysis.pdf
  confidence: 0.91
  uncertainty:
    lower: 0.009
    upper: 0.013
  method: derived_from_voltage_current_operating_point
  retrieved_at: 2026-09-30
```

### 8A.1 Parameter states

Every parameter has one of the following states:

| State | Meaning | Allowed in final report |
|---|---|---|
| `measured` | Directly measured by the user | Yes |
| `datasheet` | Explicitly stated by manufacturer documentation | Yes |
| `ecalc` | Directly extracted from eCalc | Yes |
| `derived` | Calculated from known project values | Yes |
| `literature` | Obtained from a technical publication/reference | Yes |
| `web_verified` | Retrieved from a manufacturer or authoritative source | Yes |
| `estimated` | Physically/empirically estimated | Yes, with uncertainty |
| `calibrated` | Fitted against eCalc/bench observations | Yes, with calibration metrics |
| `assumed` | Temporary engineering assumption | Yes, but clearly flagged |
| `unknown` | Insufficient evidence | No silent substitution |

### 8A.2 Evidence hierarchy

The AI evidence agent must distinguish between **hard constraints**, **soft evidence**, and **model assumptions**.

**Hard constraints**
- Manufacturer maximum voltage
- Manufacturer continuous/burst current
- Datasheet winding resistance
- Datasheet motor mass
- Datasheet ESC ratings
- User-provided measured values
- Explicit eCalc inputs/results

**Soft evidence**
- Independent thrust-stand tests
- Independent temperature measurements
- Engineering papers
- Application notes
- Reputable technical reviews
- Community build logs

**Assumptions**
- Generic convection coefficients
- Estimated contact resistance
- Estimated thermal capacitance
- Estimated emissivity
- Estimated sensor lag
- Missing geometric information

Hard constraints may bound or reject a candidate model. Soft evidence may improve priors and parameter estimates but must not override a manufacturer limit. Assumptions must carry uncertainty.

### 8A.3 Search-before-assume behavior

When a required parameter is missing, the parameter resolver should automatically attempt to find it.

Example:

```text
Missing parameter:
motor winding resistance

        ↓

Search exact manufacturer + motor model

        ↓

Manufacturer datasheet / official product page found?
        ├── YES → extract value + citation
        └── NO

        ↓

Search technical databases / independent measurements

        ↓

Multiple compatible measurements found?
        ├── YES → robust estimate + uncertainty
        └── NO

        ↓

Can it be derived from available eCalc/project data?
        ├── YES → derive + uncertainty
        └── NO

        ↓

Can the physics model constrain it?
        ├── YES → bounded estimate + calibration
        └── NO

        ↓

Mark UNKNOWN
```

The system must **not invent a precise number** merely because the model requires one.

### 8A.4 Deriving missing parameters from user files

The toolkit should inspect all supplied project material before requesting manual input.

Potential sources include:

- eCalc CSV exports
- eCalc screenshots converted to structured values
- constraint-analysis spreadsheets
- MATLAB files
- MATLAB workspace exports
- PDFs
- datasheets
- technical reports
- BOM files
- wiring diagrams
- schematics
- measured logs
- flight-test logs
- bench-test logs
- configuration YAML/JSON
- previous analysis results

Examples of useful derivations:

| Missing parameter | Possible source / derivation |
|---|---|
| Motor current | eCalc operating point or \(P/V\) |
| Electrical input power | eCalc or \(V I\) |
| Motor copper loss | \(I^2R\) when winding resistance is known |
| Effective resistance | \(P_{loss}/I^2\) when loss is available |
| Required thrust | constraint analysis / aircraft equations |
| Power loading | constraint analysis |
| Propulsive efficiency | thrust, velocity and input power |
| Battery C-rate | current / capacity |
| Voltage sag | internal resistance and current, or measured voltage |
| ESC utilisation | operating current / rated current |
| Motor utilisation | operating current / rated current |
| Thermal time constant | transient temperature response |
| Thermal capacitance | \(C_{th}=\tau/R_{th}\) |
| Thermal resistance | \(\Delta T/P_{loss}\) at steady state |
| Airframe mass | eCalc or constraint-analysis input |
| Wing loading | aircraft weight / wing area |
| Cooling severity | airflow, airspeed, installation and thermal response |
| Ambient temperature | project conditions or measured logs |
| Mission load profile | eCalc operating points / mission file |

### 8A.5 Constraint-analysis integration

Constraint-analysis files are treated as a first-class source, not merely as documentation.

The system should identify quantities such as:

- \(W/S\)
- \(T/W\)
- stall speed
- cruise speed
- climb rate
- thrust requirement
- power requirement
- wing area
- aircraft mass
- drag parameters
- propulsive efficiency
- battery energy
- flight-time requirements
- transition constraints
- VTOL hover requirements

These values constrain the thermal model because they determine the operating point that the electrical system must sustain.

For example:

```text
Constraint analysis
        ↓
Required thrust / power
        ↓
eCalc operating point
        ↓
Electrical current
        ↓
Component losses
        ↓
Thermal source terms
        ↓
Thermal propagation
```

This prevents the thermal model from optimizing a configuration that is thermally efficient only because it fails to satisfy the aircraft performance constraints.

### 8A.6 Parameter confidence and uncertainty propagation

Every uncertain parameter is represented as a distribution or bounded interval rather than a single artificial value.

Examples:

```text
R_motor ~ Normal(0.011 Ω, σ)
R_contact ~ LogNormal(...)
h_convective ~ bounded distribution
C_th ~ bounded distribution
```

Monte Carlo propagation then produces:

- temperature confidence bands,
- uncertainty in propagation time,
- uncertainty in peak temperature,
- uncertainty in thermal margin,
- uncertainty in configuration ranking.

A configuration should only be declared better when the improvement is statistically distinguishable from parameter uncertainty.

---

## 8B. Automatic Model Retuning

The toolkit should not simply estimate missing parameters once. It should **retune the physics model against the user's available eCalc and experimental evidence**.

The retuning objective is:

> Find the physically plausible parameter set that best explains the supplied eCalc operating points and, when available, measured thermal/electrical data while respecting datasheet constraints and aircraft performance constraints.

### 8B.1 Calibration hierarchy

Calibration should occur in stages:

**Stage 1 — Electrical calibration**

Fit:

- resistance,
- controller loss coefficients,
- motor no-load current,
- switching-loss coefficients,
- voltage-sag parameters.

**Stage 2 — Mechanical calibration**

Fit or verify:

- propeller operating point,
- motor torque constants,
- mechanical loss,
- aerodynamic load correction.

**Stage 3 — Thermal calibration**

Fit:

- \(R_{th}\),
- \(C_{th}\),
- convection coefficient,
- contact thermal resistance,
- thermal coupling between nodes.

**Stage 4 — System calibration**

Jointly optimize the full model while preserving hard constraints.

### 8B.2 Objective function

A weighted multi-objective loss should be used:

\[
J =
w_I J_I +
w_P J_P +
w_T J_T +
w_\eta J_\eta +
w_Q J_Q +
w_C J_C +
w_R J_R
\]

where:

- \(J_I\): current prediction error
- \(J_P\): power prediction error
- \(J_T\): temperature prediction error
- \(J_\eta\): efficiency error
- \(J_Q\): thermal-loss error
- \(J_C\): constraint violation penalty
- \(J_R\): regularisation / deviation from sourced parameters

The regularisation term is important: the optimizer must not freely move a manufacturer-specified parameter merely to improve numerical fit.

### 8B.3 Hard and soft parameter bounds

Each parameter receives bounds:

```yaml
motor_winding_resistance:
  lower: 0.009
  upper: 0.013
  source: datasheet_and_measurement
  hard: true

convection_coefficient:
  lower: 5
  upper: 80
  source: literature
  hard: false
```

Hard bounds cannot be crossed by optimization.

Soft parameters may move inside their uncertainty interval when doing so improves agreement with observed data.

### 8B.4 Recommended optimization strategy

Use a staged optimizer rather than allowing an AI model to directly guess parameters.

Recommended sequence:

1. Deterministic parameter extraction
2. Symbolic/analytical derivation where possible
3. Search and evidence retrieval
4. Initial parameter distributions
5. Global optimization
6. Local optimization
7. Monte Carlo uncertainty propagation
8. Cross-validation against held-out operating points
9. Physical sanity checks
10. Final parameter freeze

Suitable algorithms include:

- Differential Evolution
- CMA-ES
- Bayesian Optimization
- least-squares / trust-region optimization
- gradient-based optimization when the model is differentiable

The AI model acts as an **evidence and orchestration layer**, not as the authority for physical constants.

### 8B.5 Preventing overfitting to eCalc

eCalc values are valuable reference data, but agreement with eCalc alone does not establish physical truth.

Therefore:

- split eCalc operating points into calibration and validation sets,
- never optimize against every point and report the same points as validation,
- preserve manufacturer limits,
- compare against independent technical documents,
- use bench data when available,
- report residuals,
- report uncertainty,
- flag cases where the model matches eCalc but violates physical constraints.

This follows the project's existing principle that eCalc agreement is a consistency check rather than proof of correctness. 

### 8B.6 Active parameter acquisition

If a parameter remains highly influential and uncertain, the system should identify the **single most useful missing measurement or document**.

Example:

```text
Parameter uncertainty:
ESC thermal resistance = high

Sensitivity:
dT/dRth = very high

Best next evidence:
Measure ESC case temperature at 40 A for 5 minutes
```

The tool can therefore produce:

```text
Recommended next measurement:
Motor winding resistance at 20 °C

Expected benefit:
Reduce predicted motor peak-temperature uncertainty by ~31%

Priority:
HIGH
```

This turns the model into an iterative engineering workflow rather than a one-shot simulator.

---

## 8C. Retuning Workflow

The complete workflow becomes:

```text
                USER PROJECT
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   eCalc CSV     Constraint      Technical
                  Analysis        Documents
       │             │             │
       └─────────────┼─────────────┘
                     ▼
             DATA EXTRACTION
                     │
                     ▼
          CANONICAL PARAMETER STORE
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   Known parameters       Missing parameters
          │                     │
          │             Evidence Retrieval
          │                     │
          │       ┌─────────────┼─────────────┐
          │       ▼             ▼             ▼
          │   Datasheets    Literature      Web
          │       │             │             │
          │       └─────────────┼─────────────┘
          │                     ▼
          │              Derivation Engine
          │                     │
          └──────────┬──────────┘
                     ▼
             INITIAL PARAMETER
                  DISTRIBUTIONS
                     │
                     ▼
             PHYSICS MODEL
                     │
                     ▼
             PARAMETER FITTING
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       eCalc       Bench     Constraints
       Error       Error      Penalty
          └──────────┼──────────┘
                     ▼
              OPTIMIZED MODEL
                     │
                     ▼
            UNCERTAINTY ANALYSIS
                     │
                     ▼
             HELD-OUT VALIDATION
                     │
                     ▼
            FINAL MODEL + REPORT
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   Thermal diagnosis     Configuration
                         optimization
```

---

## 8D. Parameter Provenance Database

The repository should maintain a machine-readable parameter registry:

```text
knowledge/
├── rag.yaml                  # RAG configuration, retrieval policy and source tiers

├── parameters/
│   ├── registry.yaml
│   ├── extracted/
│   ├── derived/
│   ├── calibrated/
│   └── uncertainty/
```

Each parameter record contains:

- canonical parameter name,
- component,
- value,
- SI unit,
- source,
- source tier,
- extraction method,
- derivation equation,
- confidence,
- uncertainty,
- date,
- document hash,
- calibration status,
- validation status.

This allows the model to answer:

> "Why is this value 0.011 Ω?"

with an auditable chain such as:

```text
0.011 Ω
  ↓
derived from user-provided motor operating point
  ↓
supported by datasheet range 0.009–0.013 Ω
  ↓
validated against eCalc current/power points
  ↓
calibrated residual = 2.8%
```

---

## 8E. AI Agent Responsibilities

The AI layer should have bounded responsibilities.

### The AI may:

- identify components,
- normalize component names,
- search for technical documentation,
- extract parameters from documents,
- compare conflicting sources,
- propose derivations,
- select suitable engineering correlations,
- identify missing parameters,
- rank evidence quality,
- generate candidate parameter ranges,
- select which parameters need calibration,
- interpret model residuals,
- recommend additional measurements.

### The AI must not:

- invent manufacturer specifications,
- override a datasheet hard limit,
- silently replace measured data,
- label an estimate as measured,
- accept a web claim without provenance,
- directly determine physical parameters without uncertainty,
- declare a configuration safe solely because an ML model predicts it is safe.

The deterministic physics and validation layer remains authoritative.

---

## 8F. Model Quality Gates

Before a tuned model is accepted, it must pass:

### Gate 1 — Data completeness

Every required parameter must be either:

- sourced,
- derived,
- estimated with uncertainty,
- calibrated,
- or explicitly marked unknown.

### Gate 2 — Source traceability

Every non-derived hard parameter must have a source reference.

### Gate 3 — Physical consistency

Verify:

- energy balance,
- current/power consistency,
- voltage consistency,
- efficiency bounds,
- temperature direction,
- thermal steady-state behavior,
- realistic time constants.

### Gate 4 — Constraint compliance

The candidate configuration must satisfy the user's:

- thrust requirement,
- stall constraint,
- power requirement,
- battery constraint,
- ESC/motor limits,
- mission requirements.

### Gate 5 — Calibration performance

Report:

- MAE,
- RMSE,
- MAPE,
- \(R^2\),
- maximum error,
- bias,
- residual distribution.

### Gate 6 — Held-out validation

A tuned model must be evaluated on operating points that were not used during calibration.

### Gate 7 — Uncertainty

The system must report whether the observed improvement is larger than the model's uncertainty.

---


## 8G. Mechanical Constraint → Electrical Loss → Thermal Root-Cause Mapping

A primary project output is not simply **which component overheats**, but **why that component was driven into the overheating condition**.

The toolkit therefore performs a causal analysis across two engineering domains:

```text
Mechanical / Aerodynamic Constraint
                ↓
Required aircraft performance
                ↓
Required thrust / power / torque
                ↓
Propulsion operating point
                ↓
Electrical current and voltage
                ↓
Component electrical losses
                ↓
Thermal generation
                ↓
Thermal propagation
                ↓
Observed / predicted overheating
```

This creates two separate but connected answers:

1. **Mechanical thermal driver:** Which aircraft performance constraint is forcing the propulsion system into the high-load operating point?
2. **Electrical thermal origin:** Which electrical component is producing the dominant heat at that operating point?

### 8G.1 Mechanical constraint identification

The system must evaluate every available constraint-analysis requirement, including where applicable:

- Stall constraint
- Cruise constraint
- Climb constraint
- Take-off constraint
- Hover constraint
- VTOL constraint
- Transition constraint
- Maximum-power constraint
- Endurance / range constraint
- Other user-defined performance constraints

For each constraint, the system determines the corresponding required operating point and its electrical consequences.

Example:

```text
Climb constraint
      ↓
Required T/W
      ↓
Required thrust
      ↓
Required shaft power
      ↓
Required motor torque
      ↓
Motor current
      ↓
Electrical losses
```

The system then compares the thermal effect of each constraint and identifies the **primary mechanical driver**.

### 8G.2 Constraint-to-thermal attribution

For each constraint \(c\), calculate a constraint-specific thermal loading score:

\[
S_c =
w_P P_{req,c}
+
w_I I_{req,c}
+
w_L L_c
+
w_T \Delta T_c
\]

where:

- \(P_{req,c}\) = electrical/shaft power associated with the constraint,
- \(I_{req,c}\) = current associated with the constraint,
- \(L_c\) = electrical loss generated by the operating point,
- \(\Delta T_c\) = predicted thermal rise attributable to that operating point.

The exact scoring weights must be configurable and should not be presented as universal engineering constants.

A more rigorous attribution should compare the thermal model under the full mission against a counterfactual scenario in which the selected constraint is relaxed or its required operating point is removed.

This produces:

```text
Constraint                    Thermal contribution
--------------------------------------------------
Stall                         4%
Cruise                       18%
Climb                        51%   ← primary driver
Take-off                     27%
```

The percentages are model-derived attribution values, not direct measurements.

### 8G.3 Electrical thermal-origin identification

Once the mechanical operating point is established, the model decomposes electrical losses by component:

\[
P_{loss,total}
=
P_{battery}
+
P_{connector}
+
P_{wiring}
+
P_{ESC}
+
P_{motor,copper}
+
P_{motor,iron}
+
P_{other}
\]

The dominant electrical source is the component contributing the largest relevant loss under the analyzed operating condition.

For example:

```text
Required thrust
      ↓
71.7 A motor current
      ↓
Motor copper loss = I²R
      ↓
Motor winding = dominant electrical heat source
```

Or:

```text
Required thrust
      ↓
High current
      ↓
ESC conduction + switching losses
      ↓
ESC power stage = dominant electrical heat source
```

The system must distinguish **heat generation** from **heat propagation**. A component may become hot because it receives heat from another component without being the original electrical source.

### 8G.4 Root-cause report

Every thermal diagnosis should include:

```text
THERMAL ROOT-CAUSE CHAIN
────────────────────────────────────────────

Mechanical driver:
    Climb constraint

Constraint:
    T/W requirement

Required performance:
    XXX N thrust

Propulsion consequence:
    XXX W / XXX Nm / XX.X A

Electrical thermal origin:
    Motor winding

Dominant loss:
    Copper I²R loss = XX W

Thermal origin node:
    N6

Propagation:
    N6 → N5 → N9

First thermal limit reached:
    ESC / Motor / Connector / Battery

Confidence:
    XX%

Primary evidence:
    eCalc + constraint analysis + datasheet
```

### 8G.5 Mechanical-vs-electrical distinction

The report must never conflate these findings.

**Mechanical cause**

> The climb constraint requires a high thrust-to-weight ratio, which forces the propulsion system to operate at high power.

**Electrical cause**

> At that operating point, motor winding copper loss is the dominant electrical heat source.

**Thermal propagation**

> The resulting motor temperature rise propagates through the modeled thermal network toward neighboring nodes.

This distinction prevents the system from incorrectly reporting "motor overheating" as the root cause when the deeper engineering cause is a performance constraint that forces excessive propulsion loading.

### 8G.6 Limiting-constraint table

The final report should contain:

| Constraint | Required performance | Current | Total loss | Peak temperature | Thermal margin | Role |
|---|---:|---:|---:|---:|---:|---|
| Stall | ... | ... | ... | ... | ... | Non-limiting |
| Cruise | ... | ... | ... | ... | ... | Moderate |
| Climb | ... | ... | ... | ... | ... | **Primary thermal driver** |
| Take-off | ... | ... | ... | ... | ... | Secondary |
| Hover | ... | ... | ... | ... | ... | Non-limiting |

The "Role" field must distinguish:

- `Performance limiting`
- `Thermally limiting`
- `Primary thermal driver`
- `Secondary thermal driver`
- `Non-limiting`
- `Unknown`

### 8G.7 Electrical heat-source table

The report should also contain:

| Component | Current | Loss | Peak temperature | Thermal margin | Contribution | Role |
|---|---:|---:|---:|---:|---:|---|
| Battery | ... | ... | ... | ... | ... | Secondary |
| Connector | ... | ... | ... | ... | ... | Low |
| Wiring | ... | ... | ... | ... | ... | Low |
| ESC | ... | ... | ... | ... | ... | Secondary |
| Motor winding | ... | ... | ... | ... | ... | **Primary heat source** |

This makes the mechanical and electrical diagnoses independently auditable.

### 8G.8 Counterfactual analysis

To establish causality rather than correlation, the system should run controlled counterfactual simulations.

Examples:

```text
Baseline:
    Full climb constraint
    → high thrust
    → high current
    → motor overheating

Counterfactual:
    Reduce required climb performance
    → lower thrust
    → lower current
    → lower motor loss
    → lower temperature
```

and:

```text
Baseline:
    High motor resistance
    → high I²R loss
    → motor overheating

Counterfactual:
    Reduce motor resistance within physically valid bounds
    → lower I²R loss
    → lower temperature
```

The difference between the baseline and counterfactual must be reported together with uncertainty.

This provides stronger evidence for the statement:

> "The climb constraint is driving the overheating."

rather than merely:

> "The climb condition and overheating occur at the same time."

### 8G.9 Output classification

The system should ultimately return a structured result such as:

```python
@dataclass
class ThermalRootCause:
    mechanical_constraint: str
    constraint_metric: str
    required_performance: float
    propulsion_operating_point: dict
    electrical_origin_component: str
    dominant_loss_mechanism: str
    dominant_loss_W: float
    thermal_origin_node: str
    propagation_path: list[str]
    first_limit_reached: str
    severity: float
    confidence: float
    evidence: list[str]
```

This output becomes part of the final report, dashboard, API and machine-readable JSON result.


## 8H. RAG and Technical Knowledge Layer

The project uses a **Retrieval-Augmented Generation (RAG)** layer as the technical knowledge and evidence-retrieval system.

The RAG layer is responsible for finding and presenting engineering evidence when the deterministic project data does not contain a required parameter. It is an **evidence retrieval and orchestration layer**, not the physics engine.

### 8H.1 Purpose of RAG

RAG is used to:

- retrieve manufacturer datasheets,
- retrieve technical manuals and application notes,
- search engineering literature,
- search previously supplied project documents,
- identify component-specific parameters,
- compare conflicting technical sources,
- provide source citations and provenance,
- propose candidate values and uncertainty ranges,
- identify missing parameters,
- recommend additional measurements,
- support the parameter-resolution agent.

RAG must not be used as the authority for deterministic engineering calculations.

### 8H.2 RAG architecture

```text
                       USER PROJECT
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          eCalc        Constraint       Technical
           CSV          Analysis        Documents
             │              │              │
             ▼              ▼              ▼
       Structured        Structured       RAG
        Parser            Parser         Ingestion
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                   PARAMETER REGISTRY
                            │
                   Missing parameter?
                       ┌────┴────┐
                      NO        YES
                       │          │
                       │          ▼
                       │     RAG RETRIEVER
                       │          │
                       │    ┌─────┼─────┐
                       │    ▼     ▼     ▼
                       │ Datasheets Papers Project
                       │           Docs
                       │    └─────┼─────┘
                       │          ▼
                       │     Reranking +
                       │     Metadata Filter
                       │          │
                       │          ▼
                       │    Evidence Pack
                       │          │
                       └──────────┤
                                  ▼
                         PARAMETER RESOLVER
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
                 Derive        Estimate      Calibrate
                    │             │             │
                    └─────────────┼─────────────┘
                                  ▼
                         PHYSICS / THERMAL
                              ENGINE
                                  │
                                  ▼
                         DIAGNOSTIC ENGINE
```

### 8H.3 Knowledge sources

The knowledge base should be divided into evidence tiers.

**Tier 1 — Project evidence**

Highest priority:

- measured data,
- eCalc exports,
- constraint-analysis outputs,
- project BOM,
- project schematics,
- project configuration files,
- flight-test logs,
- bench-test logs.

**Tier 2 — Manufacturer evidence**

- official datasheets,
- technical manuals,
- application notes,
- manufacturer specifications,
- official component documentation.

**Tier 3 — Technical literature**

- peer-reviewed papers,
- engineering handbooks,
- standards,
- university technical reports,
- aerospace/electrical engineering references.

**Tier 4 — Secondary technical sources**

- reputable engineering databases,
- independent test reports,
- verified technical measurements.

Secondary sources must not override a directly applicable manufacturer specification without an explicit conflict-resolution decision.

### 8H.4 Metadata-aware retrieval

The RAG system must not rely only on semantic similarity.

Every document chunk should carry metadata such as:

```json
{
  "component": "motor",
  "manufacturer": "A-T",
  "model": "AT3520",
  "variant": "720KV",
  "parameter": "winding_resistance",
  "unit": "ohm",
  "source_type": "datasheet",
  "source_tier": 2,
  "document_version": "...",
  "page": 4
}
```

Retrieval should combine:

1. semantic similarity,
2. exact component/model matching,
3. parameter-name matching,
4. unit compatibility,
5. source-tier priority,
6. document/version filtering,
7. applicability constraints.

This prevents a technically similar but physically different component from supplying a parameter to the model.

### 8H.5 Embeddings, vector store and reranking

The implementation may use:

- document embeddings,
- a vector database,
- metadata filtering,
- keyword/BM25 retrieval,
- hybrid retrieval,
- a cross-encoder or equivalent reranker.

The recommended retrieval sequence is:

```text
User parameter request
        ↓
Canonical parameter name
        ↓
Metadata filter
        ↓
Hybrid retrieval
        ↓
Top-K candidate documents
        ↓
Reranking
        ↓
Evidence extraction
        ↓
Conflict detection
        ↓
Evidence pack
```

The exact vector database is an implementation choice and should remain replaceable.

### 8H.6 LangChain / LangGraph orchestration

LangChain or LangGraph may be used as the **AI orchestration layer** for:

- document loading,
- retrieval pipelines,
- tool calling,
- structured extraction,
- agent state,
- evidence comparison,
- parameter-resolution workflows,
- source citation handling.

They must not replace the deterministic engineering layer.

The architecture is therefore:

```text
LangChain / LangGraph
        │
        ├── RAG retrieval
        ├── document tools
        ├── parameter-resolution agent
        ├── evidence comparison
        └── source provenance
                │
                ▼
        Parameter Resolver
                │
                ▼
        Deterministic Engineering Core
                │
        ├── constraint analysis
        ├── electrical model
        ├── thermal model
        ├── optimization
        └── uncertainty analysis
```

This separation makes the engineering calculations reproducible even if the LLM or retrieval framework is changed.

### 8H.7 Evidence pack

The RAG layer must return an evidence pack rather than an unexplained numerical answer.

Example:

```yaml
parameter: motor_winding_resistance
requested_component: AT3520 720KV

candidates:
  - value: 0.011
    unit: ohm
    source: manufacturer_document.pdf
    page: 4
    source_type: datasheet
    confidence: 0.94

  - value: 0.012
    unit: ohm
    source: independent_test.pdf
    page: 7
    source_type: technical_measurement
    confidence: 0.71

selected:
  value: 0.011
  unit: ohm
  reason: >
    Direct manufacturer value applicable to the exact motor variant.

uncertainty:
  lower: 0.010
  upper: 0.012

status: datasheet
```

The deterministic parameter resolver can then accept, reject, derive, or calibrate the candidate.

### 8H.8 Conflict resolution

When sources disagree, the system must not silently choose one.

Example:

```text
Manufacturer:       0.011 Ω
Independent test:   0.012 Ω
Derived estimate:   0.0105 Ω
```

The system should report:

```text
CONFLICT DETECTED

Preferred:
0.011 Ω

Reason:
Exact manufacturer component match.

Alternative evidence:
0.012 Ω independent measurement.

Calibration range:
0.010–0.012 Ω
```

If the disagreement materially affects the thermal prediction, the uncertainty analysis must propagate both possibilities.

### 8H.9 RAG safety boundary

RAG may propose engineering evidence, but it must never:

- invent a manufacturer specification,
- fabricate a citation,
- override a measured value without justification,
- override a hard component limit,
- silently convert an estimate into a measured value,
- use an unrelated component's specification,
- make the final thermal-safety decision.

The deterministic engineering core remains authoritative.

### 8H.10 RAG-driven missing-parameter workflow

```text
Required parameter
        ↓
Is it in project data?
   ┌────┴────┐
  YES        NO
   │          │
   │          ▼
   │      RAG retrieval
   │          │
   │      Exact component?
   │       ┌──┴──┐
   │      YES   NO
   │       │     │
   │       │   Related evidence
   │       │     │
   │       └──┬──┘
   │          ▼
   │    Evidence ranking
   │          │
   │    Can it be derived?
   │       ┌──┴──┐
   │      YES   NO
   │       │     │
   │       ▼     ▼
   │     Derive  Estimate
   │       │     │
   └───────┴─────┘
            │
            ▼
       Uncertainty
            │
            ▼
        Calibration
            │
            ▼
      Parameter registry
```

This means a user can provide incomplete project data without manually searching for every missing coefficient.

### 8H.11 RAG outputs

The RAG subsystem should expose:

- retrieved documents,
- relevant document sections,
- extracted parameter values,
- source type,
- source tier,
- citations/page numbers,
- confidence,
- uncertainty,
- conflicting values,
- applicability assessment,
- derivation suggestions,
- unresolved parameters.

These outputs are passed to the parameter-resolution layer and are also included in the final engineering report.

---

## 8. Thermal Model and Nodes

The simulation core translates an electrical load profile into transient temperatures across the thermal nodes using a coupled electrical loss model and an RC thermal network.

### 8.1 Electrical loss calculations

- **Battery (N1):** 
- **Connectors & Wires (N2, N3):** 
- **ESC MOSFETs (N5):** 
- **Motor Windings (N6):** , plus iron losses ()

### 8.2 Thermal RC network ODEs

Each node  maintains a thermal capacitance  and couples to adjacent nodes via thermal resistances , with convective cooling to ambient :

Solved using an adaptive step Runge-Kutta ODE solver (`scipy.integrate.solve_ivp`).

## 9. Fault and Stress Library

The fault library defines injectable anomalies across nodes to simulate degradation and failure states.

| Fault Name                         | Target Node | Mechanism                                                                  | Thermal Signatures                                                                 |
| ---------------------------------- | ----------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| High-resistance connector          | N2          | Contact resistance steps up by 5mΩ to 50mΩ                                 | Rapid local heating at N2, propagating to N1 and N3                                |
| MOSFET short / partial failure     | N5          | Switching loss multiplier or shorted gate                                  | Extreme temperature spike at N5, bleeding into N4                                  |
| Stator winding short (inter-turn)  | N6          | Reduced winding resistance, increased current draw                         | High core temperature at N6, dropping motor efficiency                             |
| Bearing friction / mechanical bind | N7          | Additional mechanical load torque added to shaft                           | Elevated friction heat at N7 and N6                                                |
| Airflow restriction / blockage     | N9          | Convective thermal resistance  increased by 3x                             | System-wide baseline temperature elevation                                         |
| Overloaded propeller               | N8          | Operating point beyond a component rating, from the compatibility analysis | Affected node runs hot with no component degradation. Heat scales with utilisation |

**Stress scenarios from configuration mismatch.** Besides faults, the compatibility analysis (section 7) produces *stress* cases: a healthy component operated outside its comfortable range, such as a motor run above its continuous current or an ESC with almost no headroom. These are simulated as load or cooling modifiers (current scaling, reduced airflow) instead of component degradation, and they are labelled separately. That lets the tool tell "this part is failing" apart from "this part is being misused".

## 10. Scenario Generation and Simulation

The scenario generator combines configurations, missions, and fault definitions to produce training and testing datasets.

- **Mission profiles:** Hover, waypoint transit, vertical climb, aggressive maneuvers, and loiter phases defined in `configs/missions.yaml`.
- **Environmental variations:** Ambient temperature sweeps ( to ), altitude variations, and wind speeds.
- **Fault injection timing:** Random or fixed onset times, linear or step degradation ramps, and variable severity levels.

## 11. Sensor and Noise Modeling

To bridge simulation and reality, synthetic telemetry passes through a sensor model layer:

- **Quantization:** ADC bit-depth limitations (e.g., 12-bit voltage/current sensors).
- **Additive Gaussian Noise:** Realistic sensor variance on temperature and current channels.
- **Sensor Lag:** First-order low-pass filters simulating thermal thermistor response times.
- **Packet Dropouts / Glitches:** Simulated telemetry dropouts during high-EMI conditions.

## 12. Diagnosis Engines

Four independent diagnosis engines process time-series telemetry against the healthy baseline model:

### 12.1 Engine A: Threshold Baseline

Compares sensor readings and estimated virtual node temperatures against static and dynamic thresholds. Flags immediate limit violations.

### 12.2 Engine B: Physics-Residual Bayesian Inference

Computes residuals between observed temperatures and the healthy physics simulation model. Evaluates likelihood distributions over known fault signatures using Bayesian updating.

### 12.3 Engine C: Neural Multi-Task Model

A temporal convolutional network (TCN) or Transformer encoder trained on generated scenarios to predict root-cause probabilities, origin nodes, and severity simultaneously.

### 12.4 Engine D: Hybrid Fusion

Combines outputs from Engines A, B, and C using a weighted attention gating mechanism, optimizing both rule-based safety guarantees and data-driven pattern recognition.

### 12.6 Configuration-informed priors

The compatibility analysis (section 7) gives the engines prior knowledge. If the ESC runs at 95% of its rating, then "ESC overload" and "degraded MOSFET" start with a higher prior than "connector fault". Engine B uses the flags to set prior probabilities over causes, and Engines C and D receive them as extra input features. Priors are weighted so that strong contrary evidence from the temperature data can still override them.

## 13. Visualization and Validation Outputs

All plots are generated from the simulation, the eCalc results, and the component cards. Each plot states its data source, and plots with simulated results show an **uncertainty band** from Monte Carlo runs over the thermal and loss parameters.

### 13.1 Plots

| ID     | Plot                                           | Axes and content                                                                                                                                                                                                                                                             | Answers                                                                             |
| ------ | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **P1** | **Thermal propagation curves**                 | Node temperature (or rise above ambient) against time over the mission, with mission phases shaded. Markers show the **time each node first departs from the healthy reference**, and arrows show the order of propagation                                                   | Where did heat start, how fast did it spread, and through which path?               |
| **P2** | **Electric load versus thermal losses**        | Bus current (or electrical input power) on the x-axis. Loss power per component as stacked areas on the y-axis, plus total loss and loss as a percentage of input power. Operating points (hover, cruise, full) are marked, and vertical lines show component rated currents | Which component loses the most at each load, and where do rated limits get crossed? |
| **P3** | **Temperatures versus datasheet limits**       | Steady-state and peak temperature rise per node against load, with horizontal lines at datasheet maximum temperatures from the component cards                                                                                                                               | How much thermal margin is left, and which part runs out first?                     |
| **P4** | **Configuration comparison (less-heat check)** | Baseline against alternative configurations: peak temperature per node, total heat energy over the mission, and heat per unit of thrust or per unit of energy                                                                                                                | Does the change really reduce heat?                                                 |
| **P5** | **Model validation**                           | Predicted against eCalc-reported values (current, power, efficiency, and any loss or temperature fields), as parity plots with error metrics. Bench data overlaid when available                                                                                             | Does the loss model agree with the reference values?                                |

Outputs are saved as PNG or SVG for reports and as interactive Plotly HTML for exploration.

### 13.2 Validating "less heat" for an improved configuration

An alternative configuration (for example, a different propeller, higher pack voltage, a larger ESC, or a lower-resistance connector) is checked against the baseline like this:

1. **Hold the performance fixed.** Compare at the same thrust, payload, or mission, so lower heat is not just the result of producing less power.
2. **Simulate both** with the same mission, ambient conditions, and airflow assumptions.
3. **Compute the propagation and heat metrics** below for each.
4. **Accept an improvement only if it exceeds the uncertainty band** from the Monte Carlo runs.

| Metric                | Definition                                                                             |
| --------------------- | -------------------------------------------------------------------------------------- |
| Peak rise per node    | Maximum temperature minus ambient, for each node                                       |
| Thermal margin        | Datasheet temperature limit minus peak temperature                                     |
| Total dissipated heat | Energy lost as heat over the mission (Wh)                                              |
| Loss fraction         | Total loss ÷ electrical input energy                                                   |
| Heat per unit thrust  | Dissipated heat divided by thrust delivered                                            |
| Propagation delay     | Time between a node crossing a rise threshold and its downstream neighbour crossing it |
| Propagation amplitude | Peak rise at downstream nodes relative to the source node                              |

A configuration shows *better propagation performance with less heat* when its total heat and peak rises fall, its thermal margins grow, and heat reaches downstream nodes later or with lower amplitude, all at equal performance.

## 14. Explainability and Reporting

Every diagnosis run generates a comprehensive report containing:

- Executive summary with primary cause and confidence score.
- Root-cause origin node and step-by-step propagation timeline.
- Uncertainty flag when confidence is low
- **Configuration audit:** per-component status against datasheet limits, with cited sources (section 7)
- **Plots P1 to P5** from section 13
- Actionable mitigation recommendations (e.g., upgrade ESC rating, check soldering on connector N2, increase cooling airflow).

## 15. Evaluation Harness and Metrics

The evaluation harness tests robustness across held-out scenarios using:

- **Root-Cause Isolation Accuracy:** Top-1 and Top-3 accuracy in identifying the correct failure node.
- **Onset Time Error:** Absolute time error () in detecting when degradation began.
- **Propagation Path Ordering:** Kendall’s tau rank correlation between predicted and actual propagation sequence.
- **Calibration Error:** Expected Calibration Error (ECE) for probability scores.
- Are component facts extracted correctly? Field-level precision and recall of component cards against hand-verified cards
- Are mismatches found? Precision and recall of compatibility flags on hand-labelled good and bad configurations
- Does the loss model agree with eCalc? Parity plots, MAPE, and R² for current, power, efficiency, and any loss or temperature fields in the export
- Are configurations ranked correctly by heat? Rank correlation between predicted heat and eCalc or bench ordering. Improvement claims must exceed the Monte Carlo uncertainty band

## 16. Optional Real-World Bench Validation

When measured bench or flight logs are supplied (`data/bench/`), the framework aligns timestamps, calibrates unmodeled thermal parameters using gradient descent optimization, and overlays real temperature logs onto validation plots P5.

## 17. Repository Structure

```
.
├── configs/
│   ├── system.yaml            # Thermal topology, node parameters, RC values
│   ├── faults.yaml            # Fault library definitions and signatures
│   ├── missions.yaml          # Mission profiles and load profiles
│   ├── ecalc_mapping.yaml     # eCalc CSV column mapping
│   └── compat_rules.yaml      # compatibility thresholds and rules
├── knowledge/
├── rag.yaml                  # RAG configuration, retrieval policy and source tiers

│   ├── datasheets/            # datasheet PDFs you supply
│   ├── cards/                 # generated component cards (versioned)
│   ├── cache/                 # cached search results (gitignored)
│   └── overrides.yaml         # manual corrections to extracted values
├── src/
│   ├── ingest/
│   │   ├── ecalc.py           # eCalc CSV parser
│   │   ├── mapping.py         # column mapping and unit normalisation
│   │   └── schema.py          # SystemConfig, OperatingPoint
│   ├── knowledge/
├── rag.yaml                  # RAG configuration, retrieval policy and source tiers

│   │   ├── search.py          # source search and caching
│   │   ├── extract.py         # datasheet and text extraction
│   │   ├── reconcile.py       # conflict handling
│   │   └── cards.py           # component cards
│   ├── compat/
│   │   ├── rules.py           # compatibility checks
│   │   └── stress.py          # mismatch to stress factors
│   ├── physics/
│   │   ├── electrical.py      # I^2R and switching loss models
│   │   ├── thermal_rc.py      # RC network ODE solver
│   │   └── environment.py     # Airflow and ambient coupling
│   ├── sim/
│   │   ├── generator.py       # Scenario sampler & dataset builder
│   │   └── injector.py        # Fault & stress injection engine
│   ├── sensors/
│   │   ├── noise.py           # Gaussian noise, quantization, dropouts
│   │   └── lag.py             # Thermal thermistor lag filters
│   ├── engines/
│   │   ├── threshold.py       # Engine A
│   │   ├── bayesian.py        # Engine B
│   │   ├── neural.py          # Engine C
│   │   └── hybrid.py          # Engine D
│   ├── viz/
│   │   ├── thermal_curves.py  # P1 propagation curves
│   │   ├── load_loss.py       # P2 load versus thermal loss
│   │   ├── limits.py          # P3 temperatures versus datasheet limits
│   │   ├── compare.py         # P4 configuration comparison
│   │   └── validation.py      # P5 parity plots versus eCalc and bench
│   └── eval/
│       ├── metrics.py         # Accuracy, ECE, Kendall's tau
│       └── harness.py         # Automated evaluation runner
├── tests/
│   ├── fixtures/ecalc/        # sample eCalc CSV files
│   ├── test_physics.py
│   ├── test_engines.py
│   └── test_ingest.py
├── notebooks/                 # Exploratory Jupyter notebooks
├── outputs/                   # Generated reports and figures
├── pyproject.toml
└── README.md

```

## 18. Tech Stack

| Domain                           | Tools / Libraries                                                                                                                                                                                                |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Core Language                    | Python 3.10+                                                                                                                                                                                                     |
| Numerical & Scientific           | NumPy, SciPy (ODE solvers, optimization)                                                                                                                                                                         |
| Data                             | pandas, Pydantic (configuration validation)                                                                                                                                                                      |
| Machine Learning & Probabilistic | PyTorch (Neural engine), scikit-learn, PyMC (optional Bayesian inference)                                                                                                                                        |
| Visualization                    | Plotly (interactive HTML plots), Matplotlib, Seaborn                                                                                                                                                             |
| Datasheet and web evidence       | PyMuPDF or pdfplumber (PDF text and tables), httpx with a search API, rapidfuzz (component-name matching), SQLite (evidence cache), schema-constrained language-model extraction with mandatory source citations |
| Optimization & Calibration      | SciPy optimize, Differential Evolution, CMA-ES or Bayesian Optimization, uncertainty propagation | Testing & Quality                | Pytest, Black, Ruff, MyPy                                                                                                                                                                                        |

## 19. Quick Start

> Target interface. Not implemented yet.

Bash

```
# 1. Install
pip install -e .

# 2. Ingest eCalc CSV files into typed configurations
thermavolt ingest --ecalc data/ecalc/*.csv

# 3. Build component knowledge (your datasheets plus optional web search)
thermavolt knowledge build --configs configs/ingested/ --datasheets knowledge/
├── rag.yaml                  # RAG configuration, retrieval policy and source tiers
datasheets/ --web

# 4. Audit a configuration for mismatches
thermavolt audit --config configs/ingested/cfg_001.json --out out/cfg_001_audit.md

# 5. Plot thermal propagation, load versus loss, and limit margins
thermavolt plot --config configs/ingested/cfg_001.json --plots thermal,load-loss,limits

# 6. Compare configurations (less-heat validation)
thermavolt compare --baseline cfg_001 --candidates cfg_002,cfg_003

# 7. Generate a labelled dataset from configs
thermavolt generate --config configs/experiments/baseline_dataset.yaml

# 8. Train the neural engine
thermavolt train --engine neural --config configs/experiments/neural.yaml

# 9. Evaluate all engines on held-out scenarios
thermavolt evaluate --engines threshold,bayesian,neural,hybrid

# 10. Diagnose a log file
thermavolt diagnose --input logs/run_017.csv --engine hybrid --report out/run_017.md

# 11. Launch the dashboard
thermavolt dashboard

```

## 19A. Automatic Retuning Commands

The target CLI should support automatic parameter discovery and calibration:

```bash
# Inspect all supplied project evidence
thermavolt parameters discover \
  --ecalc data/ecalc/ \
  --constraints data/constraints/ \
  --docs knowledge/
├── rag.yaml                  # RAG configuration, retrieval policy and source tiers
datasheets/ \
  --reports data/reports/

# Resolve missing parameters using evidence, derivation and bounded estimates
thermavolt parameters resolve \
  --config configs/ingested/cfg_001.json \
  --search \
  --uncertainty

# Calibrate the physics model against eCalc
thermavolt tune \
  --config configs/ingested/cfg_001.json \
  --reference data/ecalc/ \
  --method differential-evolution \
  --validate held-out

# Include bench data when available
thermavolt tune \
  --config configs/ingested/cfg_001.json \
  --reference data/ecalc/ \
  --bench data/bench/ \
  --method hybrid \
  --validate held-out

# Show why each final parameter has its value
thermavolt parameters explain \
  --config configs/ingested/cfg_001.json

# Find the most valuable missing measurement
thermavolt parameters recommend-measurement \
  --config configs/ingested/cfg_001.json
```

The commands above are target interfaces until implemented.

## 20. Roadmap

- [ ] **Phase 0: Foundations.** Finalise thermal topology, parameter ranges, and fault library. Collect sample eCalc CSV files and confirm their layout.
- [ ] **Phase 1: eCalc ingestion.** Mapping file, parser, unit normalisation, schema validation, regression fixtures.
- [ ] **Phase 2: Component knowledge.** Datasheet extraction, source search and caching, reconciliation, component cards, review workflow.
- [ ] **Phase 3: Compatibility analysis.** Rule set, status classification, mismatch-to-stress mapping, configuration audit report.
- [ ] **Phase 4: Simulator.** Electrical loss model, thermal RC network, mission profiles from eCalc operating points, healthy reference. Physics sanity tests (energy balance, steady state, known cases).
- [ ] **Phase 4C: RAG knowledge layer.** Build document ingestion, embeddings, hybrid retrieval, metadata filtering, reranking, evidence packs, provenance and conflict resolution for missing engineering parameters.
- [ ] **Phase 4D: Agent orchestration.** Use LangChain/LangGraph or a replaceable orchestration layer for retrieval, structured extraction, parameter resolution and tool calls while keeping the physics core deterministic.
- [ ] **Phase 4A: Automatic parameter resolution.** Extract parameters from eCalc, constraint-analysis files, datasheets and technical documents; derive missing values and attach provenance/uncertainty.
- [ ] **Phase 4B: Automatic model retuning.** Fit uncertain electrical and thermal parameters against eCalc and optional bench data using bounded global/local optimization and held-out validation.
- [ ] **Phase 5A: Causal attribution.** Map each aircraft performance constraint to required propulsion loading, electrical loss, thermal source, and propagation using counterfactual simulations.
- [ ] **Phase 5: Visualization.** Plots P1 to P5, uncertainty bands, configuration comparison (less-heat check).
- [ ] **Phase 6: Data.** Scenario sampler, fault and stress injection, sensor model, dataset builder, hard test splits.
- [ ] **Phase 7: Baselines.** Threshold engine and physics-residual Bayesian engine with configuration-informed priors.
- [ ] **Phase 8: Learning.** Neural multi-task engine, calibration, hybrid fusion.
- [ ] **Phase 9: Explainability and UI.** Reports, propagation graph, dashboard.
- [ ] **Phase 10: Evaluation.** Full metrics, robustness sweeps, ablations, multiple seeds.
- [ ] **Phase 10B: RAG retrieval evaluation.** Evaluate retrieval precision/recall, exact-component matching, source-tier correctness, parameter extraction accuracy, citation accuracy, conflict detection and resistance to unrelated-component retrieval.
- [ ] **Phase 10A: Causal attribution evaluation.** Evaluate constraint-driver identification, electrical heat-source identification, counterfactual consistency, and attribution stability under parameter uncertainty.
- [ ] **Phase 11 (optional): Bench validation.** Real low-power test data and parameter calibration.

**Stretch goals**

- Real-time streaming telemetry diagnosis via MQTT/WebSocket.
- Automated closed-loop PID thermal throttling recommendations.
- Interactive web-based 3D thermal propagation viewer.

## 21. Risks and Limitations

| Risk                                                                                                                                                         | Mitigation                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| **Model mismatch.** Simplified RC thermal networks may miss 3D spatial thermal gradients inside complex motor stators or ESC enclosures.                     | Calibrate parameters against empirical bench test data; model lumped nodes conservatively with safety margins.           |
| **False positives from environmental shifts.** Ambient temperature changes or sudden wind gusts can mimic internal component degradation.                    | Incorporate real-time ambient temperature sensing and relative airflow normalization into feature inputs.                |
| **Propagation ambiguity.** Fast electrical faults can cause simultaneous voltage/current drops across multiple nodes, making root-cause isolation difficult. | Rely on temporal lag analysis and probabilistic Bayesian multi-hypothesis ranking rather than single-threshold triggers. |
| **eCalc export differences.** Column layout may vary by calculator type or version.                                                                          | Column-mapping file, sample fixtures, unparseable rows reported instead of dropped, unknown values kept as unknown       |
| **Datasheet extraction errors.** Numbers misread from PDFs.                                                                                                  | Deterministic extraction first, every value tied to a source snippet, manual override file, review queue for conflicts   |
| **Unreliable web evidence.** Forum and review claims can be anecdotal, outdated, or biased.                                                                  | Tiered source trust, anecdotes labelled as such, cited links with retrieval dates, never overriding datasheet limits     |
| **Circular validation.** Agreeing with eCalc shows consistency, not truth.                                                                                   | Independent datasheet limits, optional bench data, and explicit wording in results                                       |
| **Content and terms of use.** Retrieved pages may be copyrighted or restrict automated access.                                                               | Respect `robots.txt` and site terms, store links and short cited excerpts only, cache with timestamps                    |

## 21A. Retuning Safety and Evidence Policy

Automatic tuning is deliberately constrained. The optimizer may improve parameters that are uncertain, but it must preserve hard engineering facts.

In particular:

- A manufacturer maximum current/voltage/temperature is a constraint, not a tunable target.
- A measured project value is preferred over a generic estimate.
- An eCalc result is treated as a reference observation, not unquestionable ground truth.
- A parameter inferred from insufficient evidence remains `unknown`.
- Web search is used to locate evidence, not to manufacture missing facts.
- Every tuned parameter must retain its original value, source, fitted value, bounds, uncertainty and calibration error.
- The final report must distinguish **measured**, **sourced**, **derived**, **estimated**, and **calibrated** parameters.

This policy is essential because a numerically accurate model can still be physically wrong if it has been over-fitted or allowed to violate known engineering limits.

## 22. Contributing and License

Contributions, bug reports, and pull requests are welcome. Please ensure all code changes pass unit tests (`pytest`) and adhere to formatting guidelines (`black`, `ruff`).

*Working name for the repository:* `thermal-fault-diagnosis-toolkit` (`thermavolt`)
