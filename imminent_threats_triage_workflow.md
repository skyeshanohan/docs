
sequenceDiagram
    participant SOC as SOC
    participant SA as Security Architect
    participant Team as Affected Team
    participant Tracking as Tracking & Completion

    Note over SOC: Initial State: **Monitoring**

    alt Automated Findings (Webhooks)
        SOC->>SOC: 1a Finding Detection<br/>(Wiz, SAST/SCA, Secrets, Agents)
        Note over SOC: State: **Finding Identified**
    else Manual Findings
        SOC->>SOC: 1b Manual Entry<br/>(Pentest Findings, Bug Bounty Submissions)
        Note over SOC: State: **Finding Identified**
    end

    SOC->>SOC: 2 Criteria Evaluation<br/>(CVSS ≥ 9.0 AND EPSS ≥ 0.7 AND<br/>Externally Accessible OR Active Exploitation)
    Note over SOC: authenticationRequest signature verified

    alt Criteria Met
        SOC->>SOC: 3 Auto-Create Jira Ticket<br/>(Project: SOC Managed, Link to source alert,<br/>Labels: imminent-threat, source-[tool], vertical-[team])
        Note over SOC: State: **Ticket Created**
        
        SOC->>SA: 4 Assign Security Architect<br/>(Based on vertical/team)
        Note over SA: State: **Ticket Assigned**

        SA->>Team: 5 Engage with Affected Team
        Note over Team: State: **Team Engaged**

        Team->>SA: 6 Team Provides Context<br/>(Impact Assessment)
        Note over SA: State: **Context Received**

        SA->>Team: 7 Determine Action Plan
        Note over Team: State: **Action Plan Required**

        Team->>Team: 8 Action Plan Options:<br/>• Remediation<br/>• Mitigating Control<br/>• Exception<br/>• Risk Acceptance
        Note over Team: Created response is a URL<br/>with action plan encoded

        Team->>Tracking: 9 Track Implementation<br/>(Update ticket status, Document progress,<br/>Monitor completion)
        Note over Tracking: State: **Implementation Tracking**

        Tracking->>Tracking: 10 Action Plan Complete?
        Note over Tracking: action plan signature verified

        alt Complete
            Tracking->>Tracking: 11 Close Ticket<br/>(Document final status, Archive for audit)
            Note over Tracking: State: **Process Complete**
        else Not Complete
            Tracking->>Tracking: No - Continue Monitoring
            Note over Tracking: State: **Monitoring Implementation**
        end
    else Criteria Not Met
        SOC->>SOC: End Process - No Ticket Created
        Note over SOC: State: **Monitoring** (Return to initial state)
    end
