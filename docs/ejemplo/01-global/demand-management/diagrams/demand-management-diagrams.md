# Demand Management Process - Visual Diagrams

> **Version:** 1.1  
> **Date:** 2025-03-21  
> **Author:** IT Architecture Team  
> **Purpose:** Visual representation of the demand management process end-to-end  
> **Changelog:** v1.1 - Fixed Mermaid syntax for GitHub/GitLab compatibility (removed HTML tags and special characters)

---

## Table of Contents

1. [Complete End-to-End Process](https://claude.ai/chat/4aeb5a6e-5a42-4b83-b5aa-5aa1e8ae546c#1-complete-end-to-end-process)
2. [Intake and Triage Process](https://claude.ai/chat/4aeb5a6e-5a42-4b83-b5aa-5aa1e8ae546c#2-intake-and-triage-process)
3. [RICE Scoring Decision Tree](https://claude.ai/chat/4aeb5a6e-5a42-4b83-b5aa-5aa1e8ae546c#3-rice-scoring-decision-tree)
4. [Kanban Workflow](https://claude.ai/chat/4aeb5a6e-5a42-4b83-b5aa-5aa1e8ae546c#4-kanban-workflow)
5. [Stakeholder Communication Flow](https://claude.ai/chat/4aeb5a6e-5a42-4b83-b5aa-5aa1e8ae546c#5-stakeholder-communication-flow)
6. [Weekly Backlog Grooming](https://claude.ai/chat/4aeb5a6e-5a42-4b83-b5aa-5aa1e8ae546c#6-weekly-backlog-grooming)
7. [Escalation Process](https://claude.ai/chat/4aeb5a6e-5a42-4b83-b5aa-5aa1e8ae546c#7-escalation-process)

---

## 1. Complete End-to-End Process

This diagram shows the complete demand management process from request submission to completion.

```mermaid
graph TD
    Start([Stakeholder Submits Request]) --> Intake[1. INTAKE: Email or Jira Service Desk]

    Intake --> Triage[2. INITIAL TRIAGE: Architect reviews within 3 days]

    Triage --> Valid{Valid Request?}

    Valid -->|No| Reject[Reject with Explanation]
    Reject --> NotifyReject[Notify Stakeholder]
    NotifyReject --> End1([End])

    Valid -->|Yes| Scoring[3. RICE SCORING: Calculate RICE]

    Scoring --> Priority[4. ASSIGN PRIORITY: P0/P1/P2/P3]

    Priority --> Backlog[5. ADD TO BACKLOG: Sorted by RICE Score]

    Backlog --> Groom{6. WEEKLY BACKLOG GROOMING}

    Groom --> Ready[Move to READY: 5-8 items]

    Ready --> WIP{WIP Limit Available?}

    WIP -->|No| Wait[Wait in READY Queue]
    Wait --> Groom

    WIP -->|Yes| InProgress[7. IN PROGRESS: Max 3 items]

    InProgress --> Work[Execute Work]

    Work --> Review[8. PEER REVIEW: Quality check]

    Review --> QA{Quality Pass?}

    QA -->|No| Rework[Return to In Progress]
    Rework --> Work

    QA -->|Yes| DoD{Definition of Done Met?}

    DoD -->|No| Complete[Complete Missing Items]
    Complete --> Review

    DoD -->|Yes| Done[9. DONE: Upload and Close]

    Done --> Notify[10. NOTIFY STAKEHOLDER]

    Notify --> Metrics[Update Metrics]

    Metrics --> End2([End])

    style Start fill:#e1f5ff
    style Intake fill:#fff4e1
    style Scoring fill:#ffe1f5
    style Priority fill:#f5e1ff
    style InProgress fill:#e1ffe1
    style Done fill:#d1fae5
    style End2 fill:#e1f5ff
```

---

## 2. Intake and Triage Process

Detailed view of the initial request handling.

```mermaid
graph TD
    A[Request Received] --> B{Source?}

    B -->|Email| C[Email: architecture-requests]
    B -->|Jira| D[Jira Service Desk Request Form]

    C --> E[Auto-create Jira Ticket]
    D --> E

    E --> F[Assign to Architecture Team]

    F --> G[Architect Reviews within 3 days]

    G --> H{Complete Information?}

    H -->|No| I[Request More Info from Stakeholder]
    I --> J{Response Received?}
    J -->|No after 7 days| K[Close as Incomplete]
    J -->|Yes| G

    H -->|Yes| L{In Scope for Architecture Team?}

    L -->|No| M[Redirect to Appropriate Team]
    M --> N[Notify Stakeholder]

    L -->|Yes| O{Similar Request Already Exists?}

    O -->|Yes| P[Link to Existing Request]
    P --> Q[Notify Stakeholder]

    O -->|No| R[Proceed to RICE Scoring]

    R --> S[Gather Scoring Information]

    S --> T{Stakeholder Input Needed?}

    T -->|Yes| U[Schedule 15-min Call]
    U --> V[Confirm Values]

    T -->|No| V
    V --> W[Calculate RICE Score]

    style A fill:#e1f5ff
    style W fill:#e1ffe1
```

---

## 3. RICE Scoring Decision Tree

How to calculate each component of RICE.

```mermaid
graph TD
    Start([RICE Scoring]) --> Reach{REACH: How many affected?}

    Reach -->|1-10 users/plants| R1[Reach = 5]
    Reach -->|10-25 users/plants| R2[Reach = 15]
    Reach -->|25-50 users/plants| R3[Reach = 35]
    Reach -->|50-100 users/plants| R4[Reach = 75]
    Reach -->|100+ users/plants| R5[Reach = 100]

    R1 --> Impact{IMPACT: How much help?}
    R2 --> Impact
    R3 --> Impact
    R4 --> Impact
    R5 --> Impact

    Impact -->|Minimal improvement| I1[Impact = 1]
    Impact -->|Noticeable improvement| I2[Impact = 2]
    Impact -->|Significant improvement| I3[Impact = 3]
    Impact -->|Major improvement| I4[Impact = 4]
    Impact -->|Critical/Transformative| I5[Impact = 5]

    I1 --> Confidence{CONFIDENCE: How certain?}
    I2 --> Confidence
    I3 --> Confidence
    I4 --> Confidence
    I5 --> Confidence

    Confidence -->|Speculative, no validation| C1[Confidence = 50%]
    Confidence -->|Some validation| C2[Confidence = 80%]
    Confidence -->|Validated, clear requirements| C3[Confidence = 100%]

    C1 --> Effort{EFFORT: Person-days needed?}
    C2 --> Effort
    C3 --> Effort

    Effort -->|1-3 days| E1[Effort = 2]
    Effort -->|3-10 days| E2[Effort = 7]
    Effort -->|10-30 days| E3[Effort = 20]
    Effort -->|30-60 days| E4[Effort = 45]
    Effort -->|60+ days| E5[Effort = 80]

    E1 --> Calc[Calculate RICE Score]
    E2 --> Calc
    E3 --> Calc
    E4 --> Calc
    E5 --> Calc

    Calc --> Priority{RICE Score}

    Priority -->|>= 100| P0[P0 - Critical: Start within 2 hours]
    Priority -->|50-99| P1[P1 - High: Start within 3 days]
    Priority -->|20-49| P2[P2 - Medium: Start within 2 weeks]
    Priority -->|< 20| P3[P3 - Low: Backlog, no commitment]

    P0 --> Backlog[Add to Prioritized Backlog]
    P1 --> Backlog
    P2 --> Backlog
    P3 --> Backlog

    style Start fill:#e1f5ff
    style Calc fill:#ffe1f5
    style P0 fill:#fee2e2,color:#991b1b
    style P1 fill:#fef3c7,color:#92400e
    style P2 fill:#dbeafe,color:#1e40af
    style P3 fill:#f3f4f6,color:#4b5563
    style Backlog fill:#e1ffe1
```

---

## 4. Kanban Workflow

Visual representation of the Kanban board states and transitions.

```mermaid
graph LR
    A[BACKLOG: 20+ items sorted by RICE] -->|Weekly Grooming| B[READY: 5-8 items, DoR met]

    B -->|Pull when WIP < 3| C[IN PROGRESS: Max 3 items, WIP LIMIT]

    C -->|Work Complete| D[REVIEW: 2-3 items, Peer review]

    D -->|DoD Met| E[DONE: Delivered and Documented]

    D -->|Issues Found| C

    style A fill:#9ca3af,color:#fff
    style B fill:#00a896,color:#fff
    style C fill:#02c39a,color:#fff
    style D fill:#f59e0b,color:#fff
    style E fill:#10b981,color:#fff

    classDef wip fill:#fef3c7,stroke:#f59e0b,stroke-width:4px
    class C wip
```

### Kanban Rules

```mermaid
graph TD
    Start([New Item in Backlog]) --> Sort[Sort by RICE Score]

    Sort --> Groom{Weekly Backlog Grooming?}

    Groom -->|Yes| Ready{Meets Definition of Ready?}

    Ready -->|No| Clarify[Get missing info from stakeholder]
    Clarify --> Sort

    Ready -->|Yes| Move[Move to READY: Keep 5-8 items]

    Move --> WIP{WIP Limit < 3?}

    WIP -->|No| Wait[Wait in READY]
    Wait --> Daily{Daily Standup}
    Daily --> WIP

    WIP -->|Yes| Pull[Team member PULLS item to IN PROGRESS]

    Pull --> Work[Execute Work]

    Work --> Complete{Work Complete?}

    Complete -->|No| Work

    Complete -->|Yes| PeerReview[REVIEW: Peer validation]

    PeerReview --> QA{Quality Check?}

    QA -->|Issues| Fix[Return to IN PROGRESS]
    Fix --> Work

    QA -->|Pass| DoD{All DoD Criteria Met?}

    DoD -->|No| MissingItems[Complete missing items]
    MissingItems --> PeerReview

    DoD -->|Yes| Done[Move to DONE, Notify stakeholder]

    Done --> Metrics[Update Metrics]

    Metrics --> End([Process Complete])

    style Pull fill:#e1ffe1
    style Done fill:#d1fae5
```

---

## 5. Stakeholder Communication Flow

How we manage stakeholder expectations and pressure.

```mermaid
graph TD
    A[Stakeholder: This is urgent!] --> B{Is it truly urgent?}

    B -->|Production down or Security breach| C[P0 - Critical: Start immediately]

    B -->|Business deadline, Major impact| D[Calculate RICE Score objectively]

    D --> E{RICE Score?}

    E -->|>= 100| C
    E -->|50-99| F[P1 - High: Start within 3 days]
    E -->|20-49| G[P2 - Medium: Start within 2 weeks]
    E -->|< 20| H[P3 - Low: In backlog]

    F --> I{Stakeholder Pushes Back?}
    G --> I
    H --> I

    I -->|Yes| J[Use Communication Technique]

    J --> K{Which Technique?}

    K -->|Emotion| L[Let's Get Curious: Help me understand why]

    K -->|Comparison| M[Make Constraints Visible: Which item to pause?]

    K -->|Options| N[Offer Multiple Options: A/B/C]

    L --> O[Turn emotion into problem-solving]
    M --> O
    N --> O

    O --> P{Agreement Reached?}

    P -->|No| Q[Escalate to Architecture Lead]

    Q --> R[Architecture Lead reviews with stakeholder]

    R --> S{Resolution?}

    S -->|No| T[Escalate to Chief Architect]
    S -->|Yes| U[Document decision, Update priority if needed]

    P -->|Yes| U
    T --> U

    U --> V[Communicate plan to stakeholder]

    V --> W[Track in Jira, Send regular updates]

    style A fill:#fee2e2
    style C fill:#fee2e2,color:#991b1b
    style L fill:#dbeafe
    style M fill:#dbeafe
    style N fill:#dbeafe
    style V fill:#d1fae5
```

---

## 6. Weekly Backlog Grooming

The weekly process to keep the backlog healthy.

```mermaid
graph TD
    Start([Monday 10:00 AM Backlog Grooming]) --> Review[Review all BACKLOG items]

    Review --> Sort[Sort by RICE Score]

    Sort --> Check{Each item: Still relevant?}

    Check -->|No| Archive[Archive item, Notify stakeholder]

    Check -->|Yes| Info{Complete information?}

    Info -->|No| Request[Request info from stakeholder]
    Request --> Flag[Flag for next week]

    Info -->|Yes| Score{RICE Score still accurate?}

    Score -->|No| Rescore[Recalculate RICE Score]
    Rescore --> Update[Update priority]

    Score -->|Yes| Ready{Meets Definition of Ready?}

    Ready -->|No| Missing[Identify missing information]
    Missing --> Request

    Ready -->|Yes| Move[Move top 5-8 items to READY column]

    Update --> Ready
    Archive --> Next
    Flag --> Next

    Move --> Capacity[Review team capacity this week]

    Capacity --> Plan[Plan which items move to IN PROGRESS]

    Plan --> Communicate[Send weekly update to stakeholders]

    Communicate --> End([Meeting Ends])

    style Start fill:#e1f5ff
    style Move fill:#e1ffe1
    style Communicate fill:#d1fae5
```

---

## 7. Escalation Process

When stakeholders disagree with prioritization decisions.

```mermaid
graph TD
    A[Stakeholder Disagrees with Priority] --> B[Architect Explains RICE Scoring]

    B --> C{Stakeholder Accepts?}

    C -->|Yes| D[No further action needed]

    C -->|No| E[Architect offers to recalculate with stakeholder]

    E --> F[Joint session to review scoring]

    F --> G{New RICE Score changes priority?}

    G -->|Yes| H[Update priority, Document rationale]
    H --> I[Communicate to stakeholder]

    G -->|No| J{Stakeholder still disagrees?}

    J -->|No| D

    J -->|Yes| K[Escalate to Architecture Lead]

    K --> L[Architecture Lead reviews case]

    L --> M{Business justification overrides RICE?}

    M -->|Yes| N[Create exception, Document in ADR]
    N --> O[Chief Architect approval required]

    M -->|No| P[Maintain RICE priority]

    O --> Q{Approved?}

    Q -->|Yes| H
    Q -->|No| P

    P --> R[Architecture Lead explains decision]

    R --> S{Stakeholder escalates to executive?}

    S -->|No| T[Close escalation, Maintain priority]

    S -->|Yes| U[Executive review by CTO/CIO]

    U --> V[Final decision by executive]

    V --> W{Override priority?}

    W -->|Yes| N
    W -->|No| T

    I --> End([Resolution])
    T --> End

    style A fill:#fee2e2
    style K fill:#fef3c7
    style U fill:#fef3c7
    style End fill:#d1fae5
```

---

## Process Metrics Dashboard

Key metrics to track process health:

```mermaid
graph LR
    A[Demand Management Metrics] --> B[Lead Time: Request to Done, Target < 21 days]

    A --> C[Cycle Time: In Progress to Done, Target < 7 days]

    A --> D[Throughput: Items/month, Target 8-10]

    A --> E[WIP Compliance: % time WIP <= 3, Target > 95%]

    A --> F[Priority Distribution: P0/P1/P2/P3 mix, Target Balanced]

    A --> G[Stakeholder Satisfaction: Target > 8/10]

    style A fill:#028090,color:#fff
    style B fill:#e1f5ff
    style C fill:#e1f5ff
    style D fill:#e1f5ff
    style E fill:#e1f5ff
    style F fill:#e1f5ff
    style G fill:#e1f5ff
```

---

## Summary Flow

Simple high-level view for presentations:

```mermaid
graph LR
    A[INTAKE: Single entry point] --> B[SCORE: RICE calculation]

    B --> C[PRIORITIZE: P0 to P3]

    C --> D[BACKLOG: Weekly grooming]

    D --> E[EXECUTE: WIP limit 3]

    E --> F[REVIEW: Peer validation]

    F --> G[DONE: Deliver and notify]

    style A fill:#fff4e1
    style B fill:#ffe1f5
    style C fill:#f5e1ff
    style D fill:#e1f5ff
    style E fill:#e1ffe1
    style F fill:#fff9e1
    style G fill:#d1fae5
```

---

## Legend

### Shapes

- **Rectangle**: Process step
- **Diamond**: Decision point
- **Rounded Rectangle**: Start/End
- **Parallelogram**: Input/Output (used in some diagrams)

### Colors

- 🔵 **Blue** (#e1f5ff): Information/Input
- 🟡 **Yellow** (#fff4e1, #fef3c7): Review/Decision
- 🟢 **Green** (#e1ffe1, #d1fae5): Completion/Success
- 🔴 **Red** (#fee2e2): Critical/Urgent
- 🟣 **Purple** (#f5e1ff, #ffe1f5): Calculation/Processing
- ⚫ **Gray** (#9ca3af): Backlog/Inactive

### Arrows

- **Solid arrow**: Primary flow
- **Labeled arrow**: Condition or action (e.g., "Yes", "No", "Pull when WIP < 3")

### Special Notations

- **WIP LIMIT**: Work In Progress constraints (max 3 concurrent items)
- **DoR**: Definition of Ready
- **DoD**: Definition of Done
- **RICE**: Reach × Impact × Confidence / Effort
  
  ---

**Document Control:**

- **Created**: 2025-03-21
- **Updated**: 2025-03-21 (v1.1 - GitHub compatibility fixes)
- **Format**: Mermaid diagrams (compatible with GitHub, Confluence, GitLab, VS Code)
- **Related Documents**:
  - Demand Management and Prioritization Process
  - Kanban Board Guide
  - Stakeholder Communication Playbook

---

## Troubleshooting Mermaid Rendering

### Common Issues and Solutions

**Problem: "Parse error on line X"**

- **Cause**: Special characters in labels (@, <, >, HTML tags)
- **Solution**: Use plain text without HTML tags or special characters

**Problem: Diagram doesn't render in GitHub**

- **Cause**: Mermaid syntax not supported by GitHub's version
- **Solution**: Test diagram at https://mermaid.live first
- **Tip**: Use simple labels, avoid complex formatting

**Problem: Arrows not connecting properly**

- **Cause**: Inconsistent node IDs or syntax errors
- **Solution**: Check that all node IDs match exactly (case-sensitive)

**Problem: Colors not showing**

- **Cause**: Some platforms don't support all Mermaid features
- **Solution**: Style definitions work in most renderers, but may display differently

### Best Practices for Mermaid Diagrams

1. **Keep labels short**: Use colons for additional context (e.g., "INTAKE: Single entry point")
2. **Avoid special characters**: Stick to alphanumeric + basic punctuation
3. **Test before committing**: Use https://mermaid.live to validate syntax
4. **Use consistent spacing**: Proper indentation helps readability
5. **Comment complex logic**: Add markdown notes above complex diagrams

### Alternative Export Options

If Mermaid rendering fails on your platform:

1. **Export as PNG**: Use https://mermaid.live → Edit → Export PNG
2. **Export as SVG**: Use https://mermaid.live → Edit → Export SVG
3. **Use screenshot**: Render in VS Code and take screenshot
4. **Convert to image**: Use mermaid-cli tool: `mmdc -i diagram.mmd -o diagram.png`

---
