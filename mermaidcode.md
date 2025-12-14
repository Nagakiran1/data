flowchart TB

%% ================================
%% User & Instruction Entry
%% ================================
    User(("User")) --> Instructions["Instruction Sequence"]
    Instructions --> Agency["The Agency - Main"]

%% ================================
%% Core Structure (Unchanged)
%% ================================
    Agency --> Master["Master"]

    Master --> Manual["Manual"]
    Manual --> Master

    Master --> VA["Virtual Assistant"]
    Master --> Dev["Developer"]

%% ================================
%% Flow → Actions → Assertions Additions
%% (Placed just before validation, minimal style)
%% ================================
    VA --> FlowVA["Flow Expansion"]
    FlowVA --> ActVA["Actions"]
    ActVA --> AssertVA["Assertions"]
    AssertVA --> CheckVA{"Check<br>Retries: 3"}

    Dev --> FlowDev["Flow Expansion"]
    FlowDev --> ActDev["Actions"]
    ActDev --> AssertDev["Assertions"]
    AssertDev --> CheckDev{"Check<br>Retries: 3"}

%% ================================
%% Retry Logic (Fails → Manual Observer)
%% ================================
    CheckVA -- Success --> SharedState
    CheckDev -- Success --> SharedState

    CheckVA -- "Fail 3x" --> ManualObs["Manual Observer"]
    CheckDev -- "Fail 3x" --> ManualObs

    ManualObs --> Observation
    ManualObs --> Master

%% ================================
%% Existing Supporting Agents
%% (Left exactly as your structure shows)
%% ================================
    VA --> Super
    VA --> Observation
    VA --> Safety

    Dev --> Super
    Dev --> Observation
    Dev --> Safety

    Dev --> Code
    Dev --> Browsing
    Dev --> Calendar

    Code --> Chrome
    Code --> Drivers
    Code --> Tools

%% ================================
%% Shared State & API
%% ================================
    SharedState["Shared State\nManifesto & Instructions"] --> OpenAI["OpenAI Assistants"]

%% ================================
%% Styling (Subtle, not AI-heavy)
%% ================================
    classDef user fill:#f8d6f0,stroke:#b30086,stroke-width:2px
    classDef orchestrator fill:#dff1ff,stroke:#0277bd,stroke-width:2px
    classDef agent fill:#fff2b3,stroke:#e6b800,stroke-width:1.5px
    classDef tool fill:#e1f5e0,stroke:#2e7d32,stroke-width:1.5px
    classDef state fill:#f2f2f2,stroke:#666,stroke-width:1.5px
    classDef special fill:#ffe5d9,stroke:#d35400,stroke-width:1.5px
    classDef decision fill:#fff,stroke:#000,stroke-width:1.5px
    classDef external fill:#f5e1ff,stroke:#8e44ad,stroke-width:1.5px

    User:::user
    Instructions:::state
    Agency:::orchestrator
    Master:::agent
    VA:::agent
    Dev:::agent
    Manual:::agent

    FlowVA:::state
    ActVA:::tool
    AssertVA:::state
    CheckVA:::decision

    FlowDev:::state
    ActDev:::tool
    AssertDev:::state
    CheckDev:::decision

    ManualObs:::special
    Super:::special
    Observation:::special
    Safety:::special
    SharedState:::state
    Code:::tool
    Browsing:::tool
    Calendar:::tool
    Chrome:::tool
    Drivers:::tool
    Tools:::tool
    OpenAI:::external
