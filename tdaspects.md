flowchart LR

%% ================= INCOMING QUEUE =================
Queue["📥 Incoming Segments Queue
────────────────────
• Mobile Phone (5G)
• NB-IoT
• Bluetooth
• ISM / SRD / LPD
• Emergency Broadcast
"]

%% ================= PROCESSED =================
Done2["⬅️ Processed
────────────────────
📦 Mobile Phone (4G)

✔ LTE Radio Compliance
📄 lte_radio_etsi_en_301_908.pdf

✔ Customs Clearance Certification
📄 customs_clearance_4g_device.pdf
"]

Done1["⬅️ Processed
────────────────────
📦 WiFi / Radio Module

✔ USB Type-C Receptacle
📄 usb_type_c_conformance_v1.pdf

✔ Electrical Equipment CB Scheme
📄 iec_cb_scheme_62368_cb.pdf

✔ Partial Advance Mobile Location
📄 paml_location_compliance_v2.pdf
"]

%% ================= ACTIVE SEGMENT =================
Active["📦 ACTIVE SEGMENT
────────────────────
WiFi / Radio Module

✔ USB Type-C Receptacle Connector
📄 usb_type_c_conformance_v1.pdf
Result : Compliant · Registered

✔ Electrical Equipment CB Scheme
📄 iec_cb_scheme_62368_cb.pdf
Result : Safety Approved

❌ Partial Advance Mobile Location
📄 paml_location_compliance_draft.pdf
Reason : Location accuracy < regulatory threshold
"]

%% ================= RAG PIPE (VERTICAL) =================
RAGQ["🧠 RAG Query
Segment + Regulation"]

subgraph RAG_PIPE["⬇️ RAG Decision Pipeline"]
direction TB
Loader["📥 Processor
Load Docs"]
ES["🗂 Vector DB
Elasticsearch
(Vectors + Metadata)"]
RAG["🔎 RAG Engine
Retrieve + Ground"]
LLM["🤖 In-house LLM
Reason + Decide"]
API["⚙ Backend API"]
UI["🖥 Frontend
Decision + References"]
end

%% ================= FLOWS =================
Queue --> Active
Active --> Done1 --> Done2

Active --> RAGQ
RAGQ --> Loader
Loader --> ES
ES --> RAG
RAG --> LLM
LLM --> API
API --> UI
UI --> Active

%% ================= STYLES =================
classDef queue fill:#EEF2FF,stroke:#6366F1,stroke-width:1.5px,color:#0F172A
classDef active fill:#FFF7ED,stroke:#FB923C,stroke-width:2px,color:#431407
classDef done fill:#ECFDF3,stroke:#22C55E,stroke-width:1.5px,color:#064E3B

classDef db fill:#E0F2FE,stroke:#0284C7,stroke-width:1.5px,color:#082F49
classDef rag fill:#FEF3C7,stroke:#F59E0B,stroke-width:1.5px,color:#422006
classDef llm fill:#F3E8FF,stroke:#7C3AED,stroke-width:1.5px,color:#2E1065
classDef api fill:#F1F5F9,stroke:#64748B,stroke-width:1.2px,color:#020617

class Queue queue
class Active active
class Done1,Done2 done

class Loader api
class ES db
class RAG rag
class LLM llm
class API api
class UI api
class RAGQ rag
