graph TD
    subgraph "CONTROL PLANE (GitLab)"
    A[<b>Developer</b><br>Commits COBOL Code] -->|1. Push Event| B(<b>GitLab Server</b><br>Parses .gitlab-ci.yml)
    B -->|2. Job Pending| C{<b>GitLab Runner</b><br>Linux Agent}
    C -->|3. SSH Connection<br>User: IBMUSER| D[<b>z/OS USS Session</b>]
    end

    subgraph "EXECUTION PLANE (Mainframe z/OS)"
    D -->|4. Git Sync| E[<b>USS Workspace</b><br>/u/usr/builds/app]
    
    subgraph "THE BUILD ENGINE"
        E -->|5. Invoke Script| F[<b>zAppBuild</b><br>build.groovy]
        F -->|6. Scan Source| G[<b>DBB Toolkit</b><br>Dependency Resolver API]
        G -->|7. Return Dependency Graph| F
        F -->|8. Generate JCL| H[<b>JES / MVS</b><br>Compilers & Linkers]
    end
    
    H -->|9. Write Object Deck| I[(<b>PDS Load Libs</b><br>USER.BUILD.LOAD)]
    H -->|10. Return Code (RC)| F
    F -->|11. Generate JSON Report| J[<b>BuildReport.json</b>]
    end

    J -->|12. Fetch Artifacts| C
    C -->|13. Pass/Fail Pipeline| B
