# PulseTrace Architecture  
Multi-Agent Root Cause Analysis for Data Pipeline Failures

PulseTrace uses a clean, modular, multi-agent architecture designed to mirror how real data engineers investigate pipeline failures — but automated, deterministic, and explainable.  
This document provides a deep technical overview of all architectural components.

---

# 🏗️ High-Level Architecture

<img src="../images/architecture.png" width="800">

PulseTrace contains:

- **Five specialized agents**
- **Deterministic Core Tools**
- **A2A Message Router**
- **Session Manager**
- **Memory Bank**
- **Notebook UI**
- **Optional cloud deployment**

Each component plays a precise role in the end-to-end RCA workflow.

---

# 🔹 1. Agents

PulseTrace uses five agents, each responsible for a specific investigation stage.

## **1. pulse_detector**
Purpose: Extracts failure segments from logs.  
Tools used: `log_fetch`

Outputs:
- error signatures  
- relevant log windows  

---

## **2. root_cause_diagnoser**
Purpose: Performs core RCA checks:  
- schema drift  
- invalid values  
- missing partitions  
- null / empty input checks  

Tools used:
- `schema_diff`  
- `sample_data`  

Outputs:
- structured failure signature  
- schema diff results  
- anomalous rows  

---

## **3. pattern_history_agent**
Purpose: Retrieves similar past incidents.  
Tools used: `history_query`

Outputs:
- similarity matches  
- example past RCA reports  

---

## **4. impact_scope_agent**
Purpose: Computes downstream blast radius.  
Tools used: `lineage_query`

Outputs:
- impacted tables  
- impacted pipelines  
- severity score estimate  

---

## **5. pulse_advisor**
Purpose: Synthesizes the full RCA:  
- merges agent outputs  
- prepares draft RCA  
- pauses for human approval  

After approval → triggers `save_report`.

Tools used:
- `save_report`

Outputs:
- final approved RCA  
- JSON + Markdown artifacts  

---

# 🔹 2. Deterministic Core Tools

<img src="../images/tools.png" width="500">

PulseTrace avoids hallucinations through a strict deterministic toolchain:

- **log_fetch** — extract relevant log windows  
- **schema_diff** — detect upstream schema drift  
- **sample_data** — capture invalid records  
- **lineage_query** — find downstream dependencies  
- **history_query** — search past incidents  
- **save_report** — store approved RCA  

These tools form the **zero-hallucination foundation** of PulseTrace.

---

# 🔹 3. A2A Message Router

PulseTrace uses a lightweight, in-memory A2A (Agent-to-Agent) router to ensure:  
- ordered execution  
- transparent message passing  
- reproducible investigation steps

High-level chain:

<pre>
pulse_detector
    → root_cause_diagnoser
        → pattern_history_agent
        → impact_scope_agent
            → pulse_advisor
                → (human approval)
                    → save_report
</pre>

Every message and tool call is logged for observability.

---

# 🔹 4. Session Manager

The Session Manager maintains all investigation state:

- logs  
- schema diffs  
- anomaly records  
- lineage graph  
- history matches  
- advisor drafts  

This enables full replayability and auditability of investigations.

---

# 🔹 5. Memory Bank

A long-term store containing **only approved RCAs**, enabling:

- pattern matching  
- recurrent failure detection  
- faster future investigations  

Only the Advisor can write to the Memory Bank — *after human approval*.

---

# 🔹 6. Notebook UI (Human-in-the-Loop)

<img src="../images/ui.png" width="500">

The UI allows interactive RCA:

- upload logs  
- run multi-agent RCA  
- view schema diff  
- inspect anomalies  
- View Traces
- preview draft RCA  
- download PDF  
- approve + save  

Approval flow:

1. RCA completes → draft visible  
2. User clicks **Download PDF** (optional)  
3. User checks **Approve RCA**  
4. **Save Report** activates  
5. RCA stored in Memory Bank  

Ensures **human-verified, audit-safe** RCA.

---

# 🔹 7. Cloud Deployment (Vertex AI Agent Engine)

PulseTrace includes a cloud-deployed Diagnoser agent capable of:

- schema drift detection  
- missing partition detection  
- invalid value checks  
- empty input detection  

Cloud deployment includes:

