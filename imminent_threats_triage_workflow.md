
sequenceDiagram
    participant SOC as SOC
    participant SA as Security Architect
    participant Team as Affected Team
    participant Tracking as Tracking & Completion

    Note over SOC: Initial State: Monitoring
    SOC->>SOC: 1 Finding Detection<br/>(Wiz, SAST/SCA, Secrets, Agents, Pentest)
    Note over SOC: State: Finding Identified

    SOC->>SOC: 2 Criteria Evaluation<br/>(CVSS ≥ 9.0 AND EPSS ≥ 0.7 AND<br/>Externally Accessible OR Active Exploitation)
    Note over SOC: authenticationRequest signature verified

    alt Criteria Met
        SOC->>SA: 3 Create Jira Ticket<br/>(Project: SOC Managed, Link to source alert,<br/>Labels: imminent-threat, source-[tool], vertical-[team])
        Note over SA: State: Ticket Created
    else Criteria Not Met
        SOC->>SOC: End Process
    end

    SA->>Team: 4 Assign Security Architect<br/>(Based on vertical/team)
    Note over Team: State: Architect Assigned

    SA->>Team: 5 Engage with Affected Team
    Note over Team: State: Team Engaged

    Team->>SA: 6 Team Provides Context<br/>(Impact Assessment)
    Note over SA: State: Context Received

    SA->>Team: 7 Determine Action Plan
    Note over Team: State: Action Plan Required

    Team->>Team: 8 Action Plan Options:<br/>• Remediation<br/>• Mitigating Control<br/>• Exception<br/>• Risk Acceptance
    Note over Team: Created response is a URL<br/>with action plan encoded

    Team->>Tracking: 9 Track Implementation<br/>(Update ticket status, Document progress,<br/>Monitor completion)
    Note over Tracking: State: Implementation Tracking

    Tracking->>Tracking: 10 Action Plan Complete?
    Note over Tracking: action plan signature verified

    alt Complete
        Tracking->>Tracking: 11 Close Ticket<br/>(Document final status, Archive for audit)
        Note over Tracking: State: Process Complete
    else Not Complete
        Tracking->>Tracking: No - Continue Monitoring
        Note over Tracking: State: Monitoring Implementation
    end
