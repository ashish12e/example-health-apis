# CICS Architecture Flow (Mermaid Diagram)

```mermaid
graph TD
    subgraph External_Entry_Points
        T3270["3270 Terminal"]
        Web["Web/REST Client"]
        MQ["MQ Producer"]
        Batch["Batch Job"]
        CICS_A["CICS Region (A)"]
    end

    subgraph Interfaces
        VTAM["VTAM/SNA"]
        ZCEE["z/OS Connect EE"]
        MQBridge["MQ Bridge/Listener"]
        EXCI["EXCI API"]
        MRO["MRO/IPIC/LU6.2"]
    end

    subgraph CICS_Region
        TCP["CICS TCP"]
        WebSup["CICS Web Support"]
        MQAd["CICS MQ Adapter"]
        EXCIAd["CICS EXCI Adapter"]
        CICS_B["CICS Region (B)"]
    end

    subgraph Security_and_Internal
        SecMgr["CICS Security Manager (DFHSMM)\n(RACF/ACF2/TopSecret)"]
        Tables["CICS Internal Tables\n(PCT, PPT, FCT, DCT, TCT, etc.)"]
        Task["Program Load/Execution (DFHTASK)"]
        Res["Resource Access (Files, DB2, MQ, Web, etc.)"]
        Output["Output/Response"]
    end

    T3270 --> VTAM --> TCP
    Web --> ZCEE --> WebSup
    MQ --> MQBridge --> MQAd
    Batch --> EXCI --> EXCIAd
    CICS_A -- Inter-region comms --> MRO --> CICS_B

    TCP --> SecMgr
    WebSup --> SecMgr
    MQAd --> SecMgr
    EXCIAd --> SecMgr
    CICS_B --> SecMgr

    SecMgr --> Tables --> Task --> Res --> Output
    Output -->|To originating interface| T3270
    Output -->|To originating interface| Web
    Output -->|To originating interface| MQ
    Output -->|To originating interface| Batch
    Output -->|To originating interface| CICS_A
```

---

Copy and paste this Mermaid code block into any markdown editor or tool that supports Mermaid diagrams (such as VS Code with the Mermaid extension, GitHub, or Mermaid Live Editor) to visualize the CICS architecture flow.
