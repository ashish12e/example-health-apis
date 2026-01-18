# CICS Architecture Flow

## 1. External Entry Points to CICS

- **Terminal/3270 User**  
  → VTAM/SNA  
  → CICS Terminal Control Program (TCP)  
  → Terminal-initiated Transaction

- **Web/REST API**  
  → IBM z/OS Connect EE / CICS Web Support (CWS)  
  → CICS-supplied programs: DFHWBCLI, DFHWBADX  
  → HTTP/JSON request mapped to CICS Transaction

- **MQ (IBM MQ)**  
  → MQ Bridge for CICS (DFHMQBR)  
  → MQ Listener triggers CICS Transaction

- **Batch to CICS (EXCI/START/INTRA-PARTITION TDQ)**  
  → EXCI (External CICS Interface, DFHEXCI)  
  → Batch program issues EXCI call  
  → CICS Transaction started

- **CICS-to-CICS (MRO/IPIC/LU6.2)**  
  → Inter-region communication  
  → CICS-supplied programs: DFHMRO, DFHIPIC, DFHLU62  
  → Transaction started in target region

- **Other External Interfaces**  
  - IMS Connect, DB2 SP, FTP, etc.

---

## 2. CICS Internal Processing Flow

1. **Initial Request Received**  
   Input arrives via one of the above interfaces.

2. **Security Checks**  
   RACF/ACF2/TopSecret invoked via CICS Security Manager (DFHSMM)  
   Checks:  
   - User authentication (signon, certificate, JWT, etc.)  
   - Transaction security (XTRAN, XUSER, XPROGRAM)  
   - Resource security (files, queues, DB2, etc.)

3. **Transaction Routing**  
   Transaction code (TRANID) identified  
   CICS internal tables (PCT, PPT, FCT, DCT, TCT, etc.) consulted  
   Associated program (from PCT) determined

4. **Program Load & Execution**  
   IBM-supplied program (if system transaction) or user program loaded  
   Program invoked under CICS Task Control (DFHTASK)  
   Standard CICS services available (COMMAREA, channels/containers, etc.)

5. **Resource Access**  
   File Control (DFHFCT), DB2 (DFHDB2), MQ (DFHMQ), Web (DFHWEB), etc.  
   Security checked again for resource access

6. **Response/Output**  
   Output sent back via originating interface  
   Terminal, HTTP, MQ, etc.

---

## 3. Batch-to-CICS Flow

- Batch program (COBOL/PL/I)  
  → EXCI call (DFHEXCI)  
  → CICS region receives request  
  → Security check (RACF)  
  → Transaction started  
  → Program executed  
  → Response returned to batch

---

## 4. Diagram (ASCII Art)

```
+-------------------+      +-------------------+      +-------------------+
| 3270 Terminal     |----->| VTAM/SNA          |----->| CICS TCP          |
+-------------------+      +-------------------+      +-------------------+
        |                                                    |
        v                                                    v
+-------------------+      +-------------------+      +-------------------+
| Web/REST Client   |----->| z/OS Connect EE   |----->| CICS Web Support  |
+-------------------+      +-------------------+      +-------------------+
        |                                                    |
        v                                                    v
+-------------------+      +-------------------+      +-------------------+
| MQ Producer       |----->| MQ Bridge/Listener|----->| CICS MQ Adapter   |
+-------------------+      +-------------------+      +-------------------+
        |                                                    |
        v                                                    v
+-------------------+      +-------------------+      +-------------------+
| Batch Job         |----->| EXCI API          |----->| CICS EXCI Adapter |
+-------------------+      +-------------------+      +-------------------+
        |                                                    |
        v                                                    v
+-------------------+      +-------------------+      +-------------------+
| CICS Region (A)   |<---->| MRO/IPIC/LU6.2    |----->| CICS Region (B)   |
+-------------------+      +-------------------+      +-------------------+

        |                                                    |
        v                                                    v
+-----------------------------------------------------------------------+
|                        CICS Security Manager (DFHSMM)                |
|   (RACF/ACF2/TopSecret: User, Transaction, Resource checks)           |
+-----------------------------------------------------------------------+
        |                                                    |
        v                                                    v
+-----------------------------------------------------------------------+
|   CICS Internal Tables: PCT, PPT, FCT, DCT, TCT, etc.                 |
|   (Transaction routing, program association, resource mapping)         |
+-----------------------------------------------------------------------+
        |                                                    |
        v                                                    v
+-----------------------------------------------------------------------+
|   Program Load/Execution (DFHTASK, user or IBM-supplied programs)     |
+-----------------------------------------------------------------------+
        |                                                    |
        v                                                    v
+-----------------------------------------------------------------------+
|   Resource Access (Files, DB2, MQ, Web, etc.)                         |
+-----------------------------------------------------------------------+
        |                                                    |
        v                                                    v
+-------------------+      +-------------------+      +-------------------+
| Output/Response   |<-----| Interface Adapter |<-----| CICS Application  |
+-------------------+      +-------------------+      +-------------------+
```

---

## 5. Key IBM-Supplied Programs/Modules

- DFHSMM: Security Manager
- DFHEXCI: External CICS Interface
- DFHMRO: MRO communication
- DFHIPIC: IPIC communication
- DFHLU62: LU6.2 communication
- DFHWBCLI/DFHWBADX: Web/REST support
- DFHMQBR: MQ Bridge
- DFHTASK: Task Control
- DFHFC: File Control
- DFHDB2: DB2 Adapter
- DFHWEB: Web Adapter

---

## 6. Security Flow

- All entry points funnel through CICS Security Manager (DFHSMM)
- RACF/ACF2/TopSecret invoked for:
  - User authentication
  - Transaction authorization
  - Resource access
- Security checks occur at:
  - Signon
  - Transaction start
  - Resource access (files, DB2, MQ, etc.)

---

## 7. Transaction Triggering

- After all security checks, CICS consults the PCT (Program Control Table) to associate the transaction code (TRANID) with the program
- The program is loaded and executed as a CICS task

---

## 8. Modernization/Integration Notes

- All flows can be API-enabled via z/OS Connect EE
- Security can be modernized with JWT, OAuth, SSO, etc.
- Batch-to-CICS can be modernized with REST or MQ triggers
- CICS-to-CICS can use IPIC for secure, high-performance comms

---

This flow can be adapted into a graphical diagram for documentation or presentations. If you want a visual diagram (e.g., PlantUML, Mermaid, or PNG), let me know your preferred format!
