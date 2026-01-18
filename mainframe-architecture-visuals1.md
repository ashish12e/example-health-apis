# Mainframe Architecture Visuals

This document provides visual representations of mainframe architecture, including hardware, subsystems, security, batch processing, and modernization integration. Diagrams are provided in Mermaid and ASCII art formats for easy reference and editing.

---

## 1. Mainframe Hardware and LPARs

```mermaid
graph TD
    ZHW["IBM Z Hardware (CPC)"]
    LPAR1["LPAR 1 (z/OS)"]
    LPAR2["LPAR 2 (Linux on Z)"]
    LPAR3["LPAR 3 (z/VM)"]
    DASD["DASD (Disk Storage)"]
    TAPE["Tape Storage"]
    ZHW --> LPAR1
    ZHW --> LPAR2
    ZHW --> LPAR3
    LPAR1 --> DASD
    LPAR1 --> TAPE
    LPAR2 --> DASD
    LPAR3 --> DASD
```

---

## 2. Subsystems and Component Integration

```mermaid
graph TD
    LPAR["LPAR (z/OS)"]
    JES["JES2/JES3"]
    CICS["CICS"]
    IMS["IMS"]
    DB2["DB2"]
    MQ["IBM MQ"]
    RACF["RACF"]
    BATCH["Batch Jobs (JCL)"]
    LPAR --> JES
    LPAR --> CICS
    LPAR --> IMS
    LPAR --> DB2
    LPAR --> MQ
    LPAR --> RACF
    JES --> BATCH
    BATCH --> DB2
    BATCH --> CICS
    CICS --> DB2
    CICS --> MQ
    CICS --> RACF
    DB2 --> RACF
    IMS --> DB2
    IMS --> RACF
```

---

## 3. COBOL-DB2 Batch Processing Flow

```mermaid
flowchart TD
    JCL["JCL (Job Control Language)"]
    BATCH["COBOL Batch Program"]
    DB2["DB2 Database"]
    RACF["RACF Security"]
    JCL -->|Submits| BATCH
    BATCH -->|SQL Calls| DB2
    BATCH -->|Security Check| RACF
    DB2 -->|Resource Auth| RACF
```

---

## 4. Security and Resource Validation

```mermaid
graph TD
    USER["User/Job"]
    RACF["RACF"]
    CICS["CICS"]
    DB2["DB2"]
    MQ["MQ"]
    FILES["DASD Files"]
    USER -->|Authenticate| RACF
    USER -->|Submit Job| CICS
    CICS -->|Resource Check| RACF
    CICS -->|DB2 Access| DB2
    CICS -->|MQ Access| MQ
    CICS -->|File Access| FILES
    DB2 -->|Security| RACF
    MQ -->|Security| RACF
    FILES -->|Security| RACF
```

---

## 5. Modernization and API Enablement

```mermaid
graph TD
    WEB["Web/Mobile App"]
    API["API Gateway (z/OS Connect EE)"]
    CICS["CICS"]
    DB2["DB2"]
    MQ["MQ"]
    WEB -->|REST/JSON| API
    API -->|REST/JSON| CICS
    API -->|REST/JSON| DB2
    API -->|REST/JSON| MQ
    CICS -->|Business Logic| DB2
    CICS -->|Messaging| MQ
```

---

## 6. ASCII Art: Mainframe System Overview

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
```

---

## 7. Batch Processing Flow and DASD/DB2 Dependency

### 7.1 Batch Job Flow (JCL, COBOL, DB2, DASD)

```mermaid
flowchart TD
    SUBMIT["JCL Submit"]
    JES["JES2/JES3 Scheduler"]
    BATCH["COBOL Batch Program"]
    RACF["RACF Security"]
    DB2["DB2 Database"]
    DASD["DASD (VSAM/Sequential Files)"]
    SUBMIT --> JES
    JES -->|Allocates Resources| BATCH
    BATCH -->|Security Check| RACF
    BATCH -->|SQL/EXEC CICS| DB2
    BATCH -->|File I/O| DASD
    DB2 -->|Tablespaces/Logs| DASD
    BATCH -->|Output| JES
```

### 7.2 ASCII Art: Batch, DASD, and DB2

```
+-----------+     +-----------+     +-----------+     +-----------+
|  JCL Job  | --> |  JES2/3   | --> |  COBOL    | --> |  RACF     |
+-----------+     +-----------+     +-----------+     +-----------+
                                         |
                                         v
                                   +-----------+
                                   |   DB2     |
                                   +-----------+
                                         |
                                         v
                                   +-----------+
                                   |   DASD     |
                                   +-----------+

COBOL batch jobs can access DB2 (for SQL) and DASD (for file I/O). DB2 itself stores data on DASD.
```

---

### 7.3 Key Points
- Batch jobs are submitted via JCL and managed by JES2/JES3.
- COBOL batch programs can access DB2 using embedded SQL and DASD for file operations (VSAM, sequential datasets).
- DB2 stores all tablespaces, indexes, and logs on DASD.
- RACF validates user and resource access for both DB2 and DASD.
- Resource allocation (memory, CPU, I/O) is managed by z/OS and WLM.

---

**Tip:** Copy Mermaid code blocks into a Mermaid-enabled markdown editor or VS Code extension to view diagrams interactively.

---

## 8. Modern Mainframe Offerings: USS, z/Linux, zIIP, and More

### 8.1 UNIX System Services (USS)
- USS is a POSIX-compliant UNIX environment integrated into z/OS.
- Allows running shell scripts, UNIX utilities, and open-source software (Python, Node.js, etc.) natively on z/OS.
- Enables porting of UNIX/Linux applications to the mainframe.
- Used for DevOps tooling, web servers (Apache, NGINX), and hybrid workloads.

### 8.2 z/Linux (Linux on Z)
- Full-featured Linux distributions (RHEL, SUSE, Ubuntu) run natively on IBM Z hardware, either in LPARs or under z/VM.
- Supports open-source stacks, containers, and cloud-native workloads.
- Used for web servers, application servers, databases, and integration with distributed/cloud environments.
- Enables consolidation of Linux and z/OS workloads on a single mainframe.

### 8.3 zIIP (z Integrated Information Processor)
- Special-purpose processor for offloading eligible workloads from general-purpose CPUs (CPs).
- Handles DB2 analytics, XML parsing, Java, z/OS Connect, and select network workloads.
- Reduces software licensing costs (zIIP cycles are not charged like CP cycles).
- Improves overall system throughput and cost efficiency.

### 8.4 zAAP (z Application Assist Processor)
- (Legacy, now mostly replaced by zIIP) Used to offload Java workloads from general CPUs.
- Java workloads now typically run on zIIP.

### 8.5 IFL (Integrated Facility for Linux)
- Processor dedicated to running Linux workloads on IBM Z.
- Not used for z/OS, only for Linux on Z (z/Linux) LPARs or z/VM guests.
- Reduces cost of Linux processing on mainframe.

### 8.6 z/OS Container Extensions (zCX)
- Allows running Docker-compatible Linux containers natively on z/OS.
- Enables microservices, cloud-native apps, and open-source tools to run alongside traditional workloads.
- Bridges mainframe and cloud-native development.

### 8.7 Modern Use Cases and Integration
- Hybrid cloud: Seamless integration with IBM Cloud, AWS, Azure, and on-premises systems.
- DevOps: Git, Jenkins, Ansible, and other tools run on USS or z/Linux for CI/CD pipelines.
- API enablement: z/OS Connect EE exposes mainframe assets as REST APIs for web/mobile/cloud.
- Analytics: zIIP and z/OS analytics tools enable real-time insights on transactional data.
- Security: Centralized with RACF, supports modern encryption, MFA, and compliance.

---

### 8.8 Visual: Modern Mainframe Offerings

```mermaid
graph TD
    ZHW["IBM Z Hardware"]
    LPAR1["LPAR (z/OS)"]
    LPAR2["LPAR (z/Linux)"]
    LPAR3["LPAR (z/VM)"]
    USS["UNIX System Services (USS)"]
    zIIP["zIIP Processor"]
    IFL["IFL Processor"]
    zCX["z/OS Container Extensions (zCX)"]
    ZHW --> LPAR1
    ZHW --> LPAR2
    ZHW --> LPAR3
    LPAR1 --> USS
    LPAR1 --> zCX
    LPAR1 --> zIIP
    LPAR2 --> IFL
    LPAR2 --> zIIP
    LPAR3 --> LPAR2
    zCX -->|Containers| LPAR1
    USS -->|POSIX Apps| LPAR1
    LPAR2 -->|Linux Apps| IFL
```

---

### 8.9 Summary Table

| Offering         | Purpose/Use Case                                      |
|-----------------|-------------------------------------------------------|
| USS             | UNIX environment on z/OS for scripts, open source     |
| z/Linux         | Native Linux on Z for open source, cloud, web         |
| zIIP            | Offload DB2, Java, analytics, APIs, reduce cost       |
| IFL             | Dedicated Linux processor, cost-effective Linux       |
| zCX             | Run Docker containers on z/OS, cloud-native bridge    |
| z/VM            | Virtualization, run many Linux/z/OS guests            |

---

## 9. Enterprise Environments: UNIT, IT, UAT, PROD

### 9.1 Visual: Environment Lifecycle and Promotion Flow

```mermaid
flowchart TD
    DEV["DEV (Developer Workstation)"]
    UNIT["UNIT (Unit Test LPAR)"]
    IT["IT (Integration Test LPAR)"]
    UAT["UAT (User Acceptance Test LPAR)"]
    PROD["PROD (Production LPAR)"]
    CODE["CICS-COBOL-DB2 Source Code"]
    BUILD["Build & Compile (JCL, DB2 Bind)"]
    PACKAGE["Load Module & DB2 Package"]
    MIGRATE1["Promote to UNIT"]
    MIGRATE2["Promote to IT"]
    MIGRATE3["Promote to UAT"]
    MIGRATE4["Promote to PROD"]
    CODE --> BUILD
    BUILD --> PACKAGE
    PACKAGE --> MIGRATE1
    MIGRATE1 --> UNIT
    UNIT --> MIGRATE2
    MIGRATE2 --> IT
    IT --> MIGRATE3
    MIGRATE3 --> UAT
    UAT --> MIGRATE4
    MIGRATE4 --> PROD
```

---

### 9.2 Example: CICS-COBOL-DB2 Code Promotion Steps
1. **Development:**
   - Developer writes COBOL-DB2 code and CICS maps on a workstation or in USS/z/Linux.
2. **Build:**
   - Code is compiled using JCL; DB2 precompiler generates DBRM.
   - DBRM is bound to a test plan/package in DB2.
   - Load module is created and stored in a test load library.
3. **Unit Test (UNIT):**
   - Code is tested in a dedicated LPAR with test CICS and DB2 subsystems.
   - Test datasets and DB2 tables are used.
4. **Integration Test (IT):**
   - Code is promoted to IT LPAR.
   - Integration with other applications, MQ, and external systems is tested.
5. **User Acceptance Test (UAT):**
   - Business users validate functionality in UAT LPAR.
   - Data is masked or subset of production.
6. **Production (PROD):**
   - Code is promoted to PROD LPAR.
   - Final DB2 bind to production plan/package.
   - Load module is copied to production libraries.
   - CICS resources (programs, transactions, maps) are defined in PROD CICS region.

---

### 9.3 Common Errors and Resolutions
| Error Type                | Example/Error Code         | Resolution                                    |
|---------------------------|---------------------------|-----------------------------------------------|
| DB2 Bind Error            | DSNT408I, -805            | Ensure correct DBRM/plan/package is bound and available in target DB2 subsystem. |
| Authorization Failure     | -551, NOTAUTH             | Grant required privileges in DB2 or RACF; check user/resource access.           |
| Load Module Not Found     | S806                      | Verify load library concatenation and module promotion.                        |
| CICS Program Not Defined  | AEI0, AEIQ                | Define program/transaction in CICS region (CSD update, newcopy).               |
| Data Mismatch             | SQLCODE -311, -302        | Check data types, lengths, and host variable definitions.                      |
| Environment Mismatch      | Plan/package not found    | Ensure correct plan/package is promoted and rebound in each environment.       |
| File/Dataset Not Found    | S213, S013                | Allocate required datasets; check JCL and catalog entries.                     |
| MQ Connection Error       | MQRC 2059, 2035           | Verify MQ queue manager/channel definitions and security.                      |

---

### 9.4 Additional Notes
- Automated tools (Endevor, ISPW, UrbanCode) are often used for code promotion and environment management.
- Each environment typically has its own CICS, DB2, MQ, and dataset instances for isolation.
- Data masking and refresh processes are used to protect sensitive data in non-prod environments.
- Regression and performance testing are performed before PROD promotion.
- Change management and approvals are required for PROD moves.

---

**Document Version:** 1.0  
**Date Created:** January 18, 2026  
**Purpose:** Visual Reference for Mainframe Architecture
