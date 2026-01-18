# CICS Architecture Flow - Comprehensive Guide

## Overview
This document provides a detailed flow of CICS (Customer Information Control System) architecture, including connection methods, security verification, transaction processing, and batch integration.

---

## 1. CICS CONNECTION ENTRY POINTS

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXTERNAL REQUEST SOURCES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │   3270 TERM  │  │  WEB BROWSER │  │  REST/Web Services   │   │
│  │  (Bisync)    │  │   (HTTP)     │  │  (z/OS Connect EE)   │   │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘   │
│         │                 │                      │               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │ MQ MESSAGE   │  │ CICS-to-CICS │  │  BATCH PROGRAMS      │   │
│  │  (MQSeries)  │  │  (IPIC/Link) │  │  (CICS Bridge/ExCI)  │   │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘   │
│         │                 │                      │               │
└─────────┼─────────────────┼──────────────────────┼───────────────┘
          │                 │                      │
          └─────────────────┼──────────────────────┘
                            │
                            ▼
                  ┌──────────────────────┐
                  │  CICS REGION ENTRY   │
                  │   (CICS Terminal)    │
                  └──────────────────────┘
```

---

## 2. DETAILED CICS REQUEST PROCESSING FLOW

```
START: External Request Arrives
       │
       ▼
┌──────────────────────────────────────┐
│  Step 1: RECEIVE INPUT               │
│  ─────────────────────────────────   │
│  • Terminal input received           │
│  • Web request accepted              │
│  • Batch request authenticated       │
│  • MQ message dequeued               │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│  Step 2: IBM SUPPLIED PROGRAM INVOKED│
│  ─────────────────────────────────   │
│  DFHKTC (Terminal Control Program)   │
│  - Handles terminal I/O              │
│  - Manages input buffer              │
│  - Determines transaction code       │
│  - Sets up User ID if available      │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│  Step 3: EXTRACT TRANSACTION CODE    │
│  ─────────────────────────────────   │
│  Transaction Code = First 4 chars    │
│  Example: "CLAI" (from input)        │
│  Error handling if invalid           │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│  Step 4: SECURITY VERIFICATION       │
│  ─────────────────────────────────── │
│                                      │
│  Phase 1: USER AUTHENTICATION        │
│  ┌─────────────────────────────────┐ │
│  │ IBM Program: DFHSIT/DFHZC       │ │
│  │ (System Initialization)         │ │
│  ├─────────────────────────────────┤ │
│  │ • Check if user already signed │ │
│  │ • If NO: FORCE LOGON            │ │
│  │   - Call DFHZSE (Sign-On)      │ │
│  │   - Validate User ID            │ │
│  │   - Validate Password           │ │
│  │   - Call RACF/ACF2/TopSecret   │ │
│  │   - Set TCTTE (Term Control Blk)│ │
│  │ • If YES: Continue with session │ │
│  └─────────────────────────────────┘ │
│                                      │
│  Phase 2: PCT LOOKUP (Program Ctrl) │
│  ┌─────────────────────────────────┐ │
│  │ IBM Program: DFHTX              │ │
│  │ (Transaction Router)            │ │
│  ├─────────────────────────────────┤ │
│  │ • Look up tran code in PCT      │ │
│  │ • Retrieve associated program   │ │
│  │ • Retrieve security profile     │ │
│  │ • Retrieve operator class       │ │
│  │ • If NOT FOUND: Error           │ │
│  └─────────────────────────────────┘ │
│                                      │
│  Phase 3: TRANSACTION SECURITY      │
│  ┌─────────────────────────────────┐ │
│  │ IBM Program: DFHSAP             │ │
│  │ (Security & Authorization)      │ │
│  ├─────────────────────────────────┤ │
│  │ • Check transaction access      │ │
│  │   - User Role vs Transaction    │ │
│  │   - Operator Class check        │ │
│  │   - RACF/ACF2/TopSecret validate│ │
│  │ • If NOT AUTHORIZED: Return     │ │
│  │   DFHRESP(NOTAUTH)              │ │
│  └─────────────────────────────────┘ │
│                                      │
│  Phase 4: RESOURCE SECURITY        │
│  ┌─────────────────────────────────┐ │
│  │ IBM Program: DFHRSM             │ │
│  │ (Resource Security Manager)     │ │
│  ├─────────────────────────────────┤ │
│  │ • Validate file access          │ │
│  │ • Validate program access       │ │
│  │ • Validate queue access         │ │
│  │ • Validate transient data       │ │
│  │ • Apply field-level security    │ │
│  └─────────────────────────────────┘ │
└──────────────────────────────────────┘
       │
       ▼
   SECURITY PASS?
       │
   ┌───┴───┐
   │       │
  YES     NO
   │       │
   │       ▼
   │  ┌──────────────────────────────┐
   │  │ Security Error Response      │
   │  │ • Log security violation     │
   │  │ • Send error message         │
   │  │ • Audit trail entry          │
   │  │ • Return to terminal         │
   │  └──────────────────────────────┘
   │       │
   │       ▼
   │   RETURN TO START
   │
   ▼
┌──────────────────────────────────────┐
│  Step 5: TRANSACTION CODE TRIGGERING │
│  ─────────────────────────────────── │
│                                      │
│  IBM Program: DFHTP (Task Control)   │
│  ├──────────────────────────────────┤
│  │ • Create Task Control Block      │
│  │ • Allocate storage               │
│  │ • Set up exception handlers      │
│  │ • Load user program              │
│  │ • Prepare communication area     │
│  │ • Set cursor position            │
│  │ • Set operator ID in TCTTE       │
│  └──────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│  Step 6: PROGRAM EXECUTION           │
│  ─────────────────────────────────── │
│                                      │
│  Program: CLAIMCI0 (Example)         │
│  ├──────────────────────────────────┤
│  │ • Receive data from terminal     │
│  │ • Process business logic         │
│  │ • Read/Write files (VSAM, KSDS)  │
│  │ • Call other programs            │
│  │ • Call external services         │
│  │ • Build response map             │
│  │ • Send data to terminal          │
│  └──────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│  Step 7: SEND OUTPUT                 │
│  ─────────────────────────────────── │
│                                      │
│  IBM Program: DFHKTC (Term Control)  │
│  ├──────────────────────────────────┤
│  │ • Format output (BMS/3270)       │
│  │ • Send to terminal               │
│  │ • Log transaction                │
│  │ • Update statistics              │
│  │ • Release resources              │
│  └──────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│  END: Return to Wait for Input       │
└──────────────────────────────────────┘
```

---

## 3. SECURITY VERIFICATION DETAILED FLOW

```
┌──────────────────────────────────────────────────────────────┐
│                  SECURITY VERIFICATION FLOW                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                      REQUEST RECEIVED                        │
│                            │                                 │
│                            ▼                                 │
│                  ┌──────────────────┐                        │
│                  │  USER LOGGED IN? │                        │
│                  └────────┬─────────┘                        │
│                           │                                  │
│                    ┌──────┴──────┐                           │
│                    │             │                           │
│                   NO            YES                          │
│                    │             │                           │
│                    ▼             │                           │
│    ┌─────────────────────────┐   │                           │
│    │  DFHZSE: SIGNON HANDLER │   │                           │
│    ├─────────────────────────┤   │                           │
│    │ 1. Prompt for User ID   │   │                           │
│    │ 2. Prompt for Password  │   │                           │
│    │ 3. Call RACF/ACF2/TS    │   │                           │
│    │ 4. Verify credentials   │   │                           │
│    │ 5. Set User Context     │   │                           │
│    │ 6. Build TCTTE          │   │                           │
│    │ 7. Log Sign-On          │   │                           │
│    └────────────┬────────────┘   │                           │
│                 │                │                           │
│                 ▼                ▼                           │
│         ┌──────────────────────────────┐                    │
│         │  USER SESSION ESTABLISHED    │                    │
│         │  (TCTTE created/updated)     │                    │
│         └──────────────┬───────────────┘                    │
│                        │                                    │
│                        ▼                                    │
│         ┌──────────────────────────────┐                    │
│         │  LOOKUP TRANSACTION IN PCT   │                    │
│         │  (DFHTX - Transaction Router)│                    │
│         └──────────────┬───────────────┘                    │
│                        │                                    │
│                    ┌───┴────┐                               │
│                    │        │                               │
│                  FOUND   NOT FOUND                           │
│                    │        │                               │
│                    │        ▼                               │
│                    │   ┌─────────────┐                      │
│                    │   │ Invalid Tran│                      │
│                    │   │ Error Msg   │                      │
│                    │   └─────────────┘                      │
│                    │                                        │
│                    ▼                                        │
│         ┌──────────────────────────────┐                    │
│         │  CHECK TRAN SECURITY CLASS   │                    │
│         │ (From PCT definition)        │                    │
│         └──────────────┬───────────────┘                    │
│                        │                                    │
│         ┌──────────────┴──────────────┐                     │
│         │                             │                     │
│         ▼                             ▼                     │
│  ┌────────────────┐          ┌────────────────┐             │
│  │ RACF Security  │          │  No Security   │             │
│  │ Class Check    │          │  Check Needed  │             │
│  ├────────────────┤          └────────┬───────┘             │
│  │ 1. Get User    │                   │                     │
│  │    Class/Role  │                   │                     │
│  │ 2. Get Tran    │                   │                     │
│  │    Class Req   │                   │                     │
│  │ 3. Compare     │                   │                     │
│  │ 4. Authorized? │                   │                     │
│  └────────┬───────┘                   │                     │
│           │                           │                     │
│        ┌──┴──┐                        │                     │
│        │     │                        │                     │
│       YES   NO                        │                     │
│        │     │                        │                     │
│        │     ▼                        │                     │
│        │  ┌──────────────┐            │                     │
│        │  │ Not Authorized            │                     │
│        │  │ Error Msg    │            │                     │
│        │  │ Log Audit    │            │                     │
│        │  └──────────────┘            │                     │
│        │                              │                     │
│        └──────────────┬────────────────┘                     │
│                       │                                     │
│                       ▼                                     │
│         ┌──────────────────────────────┐                    │
│         │  CHECK RESOURCE SECURITY     │                    │
│         │  (DFHRSM - Resource Sec Mgr) │                    │
│         ├──────────────────────────────┤                    │
│         │ • File access permissions    │                    │
│         │ • Program access permissions │                    │
│         │ • Queue access permissions   │                    │
│         │ • Transient data permissions │                    │
│         │ • Field-level security       │                    │
│         └──────────────┬───────────────┘                    │
│                        │                                    │
│                    ┌───┴───┐                                │
│                    │       │                                │
│               AUTHORIZED  DENIED                            │
│                    │       │                                │
│                    │       ▼                                │
│                    │   ┌──────────────┐                     │
│                    │   │ Resource Sec │                     │
│                    │   │ Denied Error │                     │
│                    │   │ Log Violation│                     │
│                    │   └──────────────┘                     │
│                    │                                        │
│                    ▼                                        │
│         ┌──────────────────────────────┐                    │
│         │  ✓ SECURITY PASSED           │                    │
│         │  PROCEED TO PROGRAM EXECUTION│                    │
│         └──────────────────────────────┘                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. TRANSACTION CODE TRIGGERING FLOW

```
┌──────────────────────────────────────────────────────────────┐
│           TRANSACTION CODE ROUTING & EXECUTION               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                   TERMINAL/REQUEST INPUT                     │
│                            │                                 │
│                            ▼                                 │
│                  ┌──────────────────────┐                    │
│                  │  EXTRACT TRAN CODE   │                    │
│                  │  (First 4 chars)     │                    │
│                  │  Example: "CLAI"     │                    │
│                  └──────────┬───────────┘                    │
│                             │                                │
│                             ▼                                │
│                  ┌──────────────────────┐                    │
│                  │  DFHTX PROGRAM       │                    │
│                  │  Transaction Router  │                    │
│                  ├──────────────────────┤                    │
│                  │ • Search PCT         │                    │
│                  │   (Program Ctrl Table)                   │
│                  │ • Find entry for CLAI│                    │
│                  │                      │                    │
│                  │ PCT Entry Contains:  │                    │
│                  │ ┌────────────────┐   │                    │
│                  │ │ • Program Name │   │                    │
│                  │ │   (CLAIMCI0)   │   │                    │
│                  │ │ • Security lvl │   │                    │
│                  │ │ • Priority     │   │                    │
│                  │ │ • Routing      │   │                    │
│                  │ │ • Wait Time    │   │                    │
│                  │ │ • Profile      │   │                    │
│                  │ └────────────────┘   │                    │
│                  └──────────┬───────────┘                    │
│                             │                                │
│                             ▼                                │
│                  ┌──────────────────────┐                    │
│                  │  SECURITY VERIFIED   │                    │
│                  │  (as per flow 3)     │                    │
│                  └──────────┬───────────┘                    │
│                             │                                │
│                             ▼                                │
│                  ┌──────────────────────┐                    │
│                  │  DFHTP PROGRAM       │                    │
│                  │  Task Control Program│                    │
│                  ├──────────────────────┤                    │
│                  │ CREATE NEW TASK:     │                    │
│                  │ • Allocate TCB block │                    │
│                  │ • Set task priority  │                    │
│                  │ • Assign user ID     │                    │
│                  │ • Set terminal ID    │                    │
│                  │ • Create TCTUA       │                    │
│                  │   (Terminal Ctrl Usr)│                    │
│                  │ • Allocate storage   │                    │
│                  │ • Load program       │                    │
│                  │ • Set entry point    │                    │
│                  │ • Initialize CWA/TWA│                    │
│                  │ • Set up COMMAREA    │                    │
│                  │ • Register exception │                    │
│                  │   handlers           │                    │
│                  └──────────┬───────────┘                    │
│                             │                                │
│                             ▼                                │
│                  ┌──────────────────────┐                    │
│                  │  IBM SUPPLIED PROGRAMS│                   │
│                  │  CALLED BY CICS      │                    │
│                  ├──────────────────────┤                    │
│                  │ • DFHPC - Program    │                    │
│                  │   Control Mgr        │                    │
│                  │ • DFHSM - Storage    │                    │
│                  │   Manager            │                    │
│                  │ • DFHEP - Event Proc │                    │
│                  │ • DFHMS - Message    │                    │
│                  │   Service            │                    │
│                  └──────────┬───────────┘                    │
│                             │                                │
│                             ▼                                │
│                  ┌──────────────────────┐                    │
│                  │ USER PROGRAM EXECUTED│                    │
│                  │   CLAIMCI0.COBOL     │                    │
│                  ├──────────────────────┤                    │
│                  │ • Receive input data │                    │
│                  │ • Validate data      │                    │
│                  │ • Process business   │                    │
│                  │   logic              │                    │
│                  │ • Read VSAM files    │                    │
│                  │ • Update databases   │                    │
│                  │ • Call APIs (via     │                    │
│                  │   z/OS Connect)      │                    │
│                  │ • Format response    │                    │
│                  │ • Send output        │                    │
│                  └──────────┬───────────┘                    │
│                             │                                │
│                             ▼                                │
│                  ┌──────────────────────┐                    │
│                  │  RETURN TO CICS      │                    │
│                  ├──────────────────────┤                    │
│                  │ • DFHSC - Scheduler  │                    │
│                  │ • Release resources  │                    │
│                  │ • Log statistics     │                    │
│                  │ • Queue next tran?   │                    │
│                  │ • Pseudo conversation│                    │
│                  │   or end session     │                    │
│                  └──────────┬───────────┘                    │
│                             │                                │
│                             ▼                                │
│                  ┌──────────────────────┐                    │
│                  │ OUTPUT TO TERMINAL   │                    │
│                  │ or Web/REST Response │                    │
│                  └──────────────────────┘                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. BATCH-TO-CICS FLOW

```
┌──────────────────────────────────────────────────────────────┐
│              BATCH PROGRAM TO CICS INTEGRATION               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────┐                 │
│  │   BATCH JOB SUBMISSION                 │                 │
│  │   (e.g., COBOL Batch Program)          │                 │
│  └────────────────┬───────────────────────┘                 │
│                   │                                          │
│                   ▼                                          │
│  ┌────────────────────────────────────────┐                 │
│  │   METHOD 1: CICS BRIDGE (ExCI)         │                 │
│  │   ─────────────────────────────────    │                 │
│  │   • Most common for batch               │                 │
│  │   • Pseudo-conversational mode          │                 │
│  │   • Uses DFHEI1 interface               │                 │
│  │   • Supported in CICS regions           │                 │
│  │   • LINK to CICS service                │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  BATCH PROGRAM CALLS:                  │                 │
│  │                                        │                 │
│  │  CALL DFHECI WITH:                     │                 │
│  │  ┌──────────────────────────────────┐  │                 │
│  │  │ 1. API = 'CLAIAPI'               │  │                 │
│  │  │    (CICS API to call)            │  │                 │
│  │  │ 2. USERID = 'BATCHUSER'          │  │                 │
│  │  │ 3. INPUT-DATA = <structure>      │  │                 │
│  │  │ 4. OUTPUT-DATA = <structure>     │  │                 │
│  │  │ 5. COMMAREASIZE = <length>       │  │                 │
│  │  │ 6. DFHRESP = <response code>     │  │                 │
│  │  └──────────────────────────────────┘  │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  CICS RECEIVES BATCH REQUEST            │                 │
│  │  ┌──────────────────────────────────┐   │                 │
│  │  │ • DFHZSE creates session for     │   │                 │
│  │  │   BATCHUSER                      │   │                 │
│  │  │ • Creates pseudo-terminal        │   │                 │
│  │  │ • Allocates TCTTE                │   │                 │
│  │  │ • Sets up task                   │   │                 │
│  │  └──────────────────────────────────┘   │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  SECURITY VERIFICATION:                │                 │
│  │                                        │                 │
│  │  ┌──────────────────────────────────┐  │                 │
│  │  │ • User: BATCHUSER               │  │                 │
│  │  │ • RACF Batch User defined?      │  │                 │
│  │  │ • Transaction Authorization      │  │                 │
│  │  │ • API Authorization              │  │                 │
│  │  │ • Resource Permissions           │  │                 │
│  │  │ • Return Security violations     │  │                 │
│  │  │   if not authorized              │  │                 │
│  │  └──────────────────────────────────┘  │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  CICS API EXECUTION:                   │                 │
│  │                                        │                 │
│  │  ┌──────────────────────────────────┐  │                 │
│  │  │ • Find the API (CLAIAPI)         │  │                 │
│  │  │ • Load associated program        │  │                 │
│  │  │ • Pass COMMAREA with input data  │  │                 │
│  │  │ • Execute program                │  │                 │
│  │  │ • Capture output data            │  │                 │
│  │  │ • Build response structure       │  │                 │
│  │  └──────────────────────────────────┘  │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  METHOD 2: MQ MESSAGE QUEUE            │                 │
│  │  ─────────────────────────────────    │                 │
│  │  • Alternative to ExCI                 │                 │
│  │  • Asynchronous processing             │                 │
│  │  • Message goes to MQ queue            │                 │
│  │  • CICS picks up from queue            │                 │
│  │  • Decouples batch from CICS           │                 │
│  │  • Better for non-real-time needs      │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  METHOD 3: FILE SHARING (VIA VSAM)     │                 │
│  │  ─────────────────────────────────    │                 │
│  │  • Batch reads/writes VSAM file        │                 │
│  │  • CICS also accesses same file        │                 │
│  │  • No direct API call                  │                 │
│  │  • Requires ENQ/DEQ management         │                 │
│  │  • Not recommended for real-time       │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  RETURN TO BATCH PROGRAM               │                 │
│  │  ┌──────────────────────────────────┐  │                 │
│  │  │ • DFHRESP returned to batch      │  │                 │
│  │  │ • Output data in COMMAREA        │  │                 │
│  │  │ • Error codes if any             │  │                 │
│  │  │ • Close pseudo-terminal session  │  │                 │
│  │  │ • Release TCTTE                  │  │                 │
│  │  └──────────────────────────────────┘  │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  BATCH PROGRAM PROCESSING:             │                 │
│  │  ┌──────────────────────────────────┐  │                 │
│  │  │ • Check DFHRESP code             │  │                 │
│  │  │ • Process returned data          │  │                 │
│  │  │ • Log results                    │  │                 │
│  │  │ • Write to output dataset        │  │                 │
│  │  │ • Continue with next request     │  │                 │
│  │  └──────────────────────────────────┘  │                 │
│  └────────────┬──────────────────────────┘                 │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────────────────────────┐                 │
│  │  JOB END                               │                 │
│  └────────────────────────────────────────┘                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6. KEY IBM SUPPLIED PROGRAMS SUMMARY

| Program | Function | When Used | Key Role |
|---------|----------|-----------|----------|
| **DFHSIP** | System Initialization Program | CICS startup | Starts CICS region, loads control tables |
| **DFHZC** | CSD Manager | Initialization | Loads Program, Transaction, File definitions |
| **DFHZSE** | Sign-On Handler | First terminal contact | Validates User ID/Password, calls RACF |
| **DFHKTC** | Terminal Control Manager | Every request | Receives/sends terminal I/O |
| **DFHTX** | Transaction Router | After terminal input | Looks up transaction in PCT |
| **DFHTP** | Task Control Program | Before program execution | Creates task, allocates resources |
| **DFHPC** | Program Control Manager | Program linking | XCTL, LINK, CALL operations |
| **DFHSAP** | Security Authorization Processor | After transaction identified | Validates transaction & resource security |
| **DFHRSM** | Resource Security Manager | Before resource access | File, program, queue security checks |
| **DFHSM** | Storage Manager | Throughout | Memory allocation/deallocation |
| **DFHMS** | Message Service | Data formatting | SEND, RECEIVE operations |
| **DFHEP** | Event Processing | Error handling | Exception handlers |
| **DFHBS** | Batch/CICS Interface | Batch calls CICS | ExCI interface processing |
| **DFHAC** | Application Control | Program control | Pseudo-conversation management |
| **DFHSC** | Scheduler | Task completion | Reschedules next transaction |

---

## 7. DETAILED SECURITY MATRIX

```
┌─────────────────────────────────────────────────────────────┐
│           SECURITY CHECKS & VERIFICATION POINTS             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  CHECK POINT 1: TERMINAL SIGNON                            │
│  ────────────────────────────────────                       │
│  Responsible Program: DFHZSE                               │
│  Verification:                                             │
│    ✓ User ID exists in RACF/ACF2/TopSecret                │
│    ✓ Password matches stored value                         │
│    ✓ Account not locked/disabled                           │
│    ✓ User not already signed on                            │
│    ✓ Terminal assigned to security group                   │
│  Failure Action: Reject signon, log violation              │
│                                                             │
│  CHECK POINT 2: TRANSACTION AVAILABILITY                   │
│  ────────────────────────────────────────                  │
│  Responsible Program: DFHTX                                │
│  Verification:                                             │
│    ✓ Transaction code exists in PCT                        │
│    ✓ Transaction not disabled                              │
│    ✓ Program is available                                  │
│    ✓ Transaction is routable                               │
│  Failure Action: DFHRESP(INVREQ), error message            │
│                                                             │
│  CHECK POINT 3: TRANSACTION AUTHORIZATION                  │
│  ────────────────────────────────────────                  │
│  Responsible Program: DFHSAP                               │
│  Verification:                                             │
│    ✓ User has required operator class                      │
│    ✓ User role matches transaction class                   │
│    ✓ RACF resource class matches                           │
│    ✓ Transaction in user's authorized list                │
│  Failure Action: DFHRESP(NOTAUTH), log security event      │
│                                                             │
│  CHECK POINT 4: PROGRAM AUTHORIZATION                      │
│  ────────────────────────────────────                      │
│  Responsible Program: DFHRSM                               │
│  Verification:                                             │
│    ✓ User can execute the program                          │
│    ✓ Program security level matches                        │
│    ✓ Program not restricted                                │
│  Failure Action: DFHRESP(NOTAUTH), terminate task          │
│                                                             │
│  CHECK POINT 5: FILE AUTHORIZATION                         │
│  ────────────────────────────────────                      │
│  Responsible Program: DFHRSM                               │
│  Verification:                                             │
│    ✓ File exists and is open                               │
│    ✓ User has READ permission                              │
│    ✓ User has WRITE permission (if updating)               │
│    ✓ Record key within user's range                        │
│  Failure Action: DFHRESP(NOTAUTH) for file operation       │
│                                                             │
│  CHECK POINT 6: QUEUE AUTHORIZATION                        │
│  ────────────────────────────────────                      │
│  Responsible Program: DFHRSM                               │
│  Verification:                                             │
│    ✓ Queue exists                                          │
│    ✓ User has ENQUEUE permission                           │
│    ✓ User has DEQUEUE permission                           │
│  Failure Action: DFHRESP(NOTAUTH) for queue operation      │
│                                                             │
│  CHECK POINT 7: EXTERNAL API/SERVICE CALL                  │
│  ────────────────────────────────────────                  │
│  Responsible Program: z/OS Connect                         │
│  Verification:                                             │
│    ✓ API endpoint accessible                               │
│    ✓ SSL/TLS certificates valid                            │
│    ✓ Authentication credentials valid                      │
│    ✓ API resource authorization                            │
│  Failure Action: Connection error, DFHRESP code            │
│                                                             │
│  CHECK POINT 8: FIELD-LEVEL SECURITY                       │
│  ─────────────────────────────────────                     │
│  Responsible Program: DFHRSM / Custom                       │
│  Verification:                                             │
│    ✓ User can access sensitive field                       │
│    ✓ Field encryption/masking rules                        │
│    ✓ Audit required for field access                       │
│  Failure Action: Suppress/mask field in output             │
│                                                             │
│  CHECK POINT 9: SESSION TIMEOUT                            │
│  ────────────────────────────────                          │
│  Responsible Program: DFHTP / Scheduler                     │
│  Verification:                                             │
│    ✓ Session not expired                                   │
│    ✓ Activity timeout not exceeded                         │
│    ✓ TCTTE valid and current                               │
│  Failure Action: Forced logoff, session termination        │
│                                                             │
│  FLOW ACROSS ALL CHECKPOINTS:                              │
│  ────────────────────────────────                          │
│    Request → Sign-on → Transaction → Authorization →      │
│    Program → File/Queue → API → Response                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. COMPLETE END-TO-END FLOW WITH ALL COMPONENTS

```
                           ┌─────────────────────┐
                           │  EXTERNAL REQUESTS  │
                           ├─────────────────────┤
                           │ • Terminal (3270)   │
                           │ • Web Browser       │
                           │ • REST API          │
                           │ • MQ Queue          │
                           │ • Batch Job         │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  DFHZC: Initialize  │
                           │  (if first time)    │
                           └──────────┬──────────┘
                                      │
                                      ▼
                     ┌─────────────────────────────┐
                     │ DFHZSE: USER AUTHENTICATION │
                     │ (Sign-on/Session Check)     │
                     └──────────┬──────────────────┘
                                │
                        ┌───────┴────────┐
                        │                │
                       PASS             FAIL
                        │                │
                        │                ▼
                        │          ┌──────────────┐
                        │          │ Reject &     │
                        │          │ Return Error │
                        │          └──────────────┘
                        │
                        ▼
            ┌───────────────────────────────┐
            │  DFHTX: LOOKUP TRANSACTION    │
            │  (Extract TRAN CODE & Search  │
            │   Program Control Table)      │
            └───────────┬───────────────────┘
                        │
                        ▼
          ┌──────────────────────────────┐
          │ DFHSAP: TRANSACTION SECURITY │
          │ (Check User Authz for Tran)  │
          └───────────┬──────────────────┘
                      │
              ┌───────┴────────┐
              │                │
             PASS             FAIL
              │                │
              │                ▼
              │         ┌─────────────────┐
              │         │ Reject Tran &   │
              │         │ Return Security │
              │         │ Error (NOTAUTH) │
              │         └─────────────────┘
              │
              ▼
  ┌──────────────────────────────────┐
  │  DFHRSM: RESOURCE SECURITY CHECK │
  │  (File/Queue/Program Authz)      │
  └───────────┬──────────────────────┘
              │
      ┌───────┴────────┐
      │                │
     PASS             FAIL
      │                │
      │                ▼
      │        ┌──────────────────┐
      │        │ Deny Resource    │
      │        │ Access Error     │
      │        └──────────────────┘
      │
      ▼
  ┌──────────────────────────────────┐
  │  DFHTP: CREATE TASK & ALLOCATE   │
  │  • Create TCB                    │
  │  • Load User Program             │
  │  • Setup COMMAREA                │
  │  • Setup Exception Handler       │
  └───────────┬──────────────────────┘
              │
              ▼
  ┌──────────────────────────────────┐
  │ USER PROGRAM EXECUTION           │
  │ (e.g., CLAIMCI0.cbl)             │
  │ • Read Terminal Input            │
  │ • EXEC CICS SYNCPOINT            │
  │ • EXEC CICS READ (via DFHRSM)   │
  │ • EXEC CICS WRITE (via DFHRSM)  │
  │ • Call External API              │
  │   (via z/OS Connect)             │
  │ • Process Response               │
  │ • Format Output Map (BMS)        │
  │ • EXEC CICS SEND MAP             │
  └───────────┬──────────────────────┘
              │
              ▼
  ┌──────────────────────────────────┐
  │ DFHMS: SEND OUTPUT TO TERMINAL   │
  │ • Format BMS Map                 │
  │ • Send to 3270 Terminal or       │
  │   HTTP Response or Queue         │
  └───────────┬──────────────────────┘
              │
              ▼
  ┌──────────────────────────────────┐
  │ DFHSC: SCHEDULER                 │
  │ • Release Task Resources         │
  │ • Update Statistics              │
  │ • Log Transaction                │
  │ • Queue Next Transaction (if any)│
  │   for Pseudo-Conversation        │
  └───────────┬──────────────────────┘
              │
              ▼
  ┌──────────────────────────────────┐
  │ OUTPUT DELIVERED & READY FOR     │
  │ NEXT INPUT                       │
  └──────────────────────────────────┘
```

---

## 9. QUICK REFERENCE: KEY DEFINITIONS

### TCTTE - Terminal Control Table Element
- Created by DFHZSE during signon
- Contains: User ID, Security Info, Terminal Characteristics
- One per terminal session

### TCTUA - Terminal Control Table User Area
- User-defined data area
- Accessible across transactions
- Used to pass data between pseudo-conversational steps

### PCT - Program Control Table
- Maps transaction codes to programs
- Defines security classes
- Sets routing and priority
- Defines wait times

### CSD - CIC Supplied Definition
- Repository of all CICS definitions
- Contains: Programs, Transactions, Files, Queues, etc.
- Loaded by DFHZC at CICS startup

### DFHRESP Codes
- DFHRESP(NORMAL) = Success
- DFHRESP(NOTAUTH) = Security violation
- DFHRESP(NOTFND) = Resource not found
- DFHRESP(INVREQ) = Invalid request

---

## 10. NOTES FOR FUTURE REFERENCE

1. **Security is Multi-Layered**: Authentication → Authorization → Resource Access
2. **All Access Goes Through DFHRSM**: Resource Security Manager validates every access
3. **Transaction Codes Are Entry Points**: They trigger specific programs through PCT
4. **Batch Integration is via ExCI**: Pseudo-terminal is created for batch users
5. **IBM Programs Handle System Functions**: User programs call EXEC CICS which invokes IBM utilities
6. **Pseudo-Conversation**: Allows multiple "exchanges" within same terminal session
7. **RACF Integration**: Modern CICS uses RACF/ACF2/TopSecret for all security
8. **API Requester Flow**: Program calls REST API via z/OS Connect (wrapper around HTTP calls)

---

**Document Version**: 1.0  
**Date Created**: January 18, 2026  
**Purpose**: CICS Architecture Reference for Development & Troubleshooting  
**Applicable To**: Enterprise CICS environments with z/OS Connect integration
