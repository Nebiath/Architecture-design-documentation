# Demand Management Process - Visual Diagrams

> **Version:** 1.0  
> **Date:** 2025-03-21  
> **Author:** IT Architecture Team  
> **Purpose:** Visual representation of the demand management process end-to-end

---

## Table of Contents

1. [Complete End-to-End Process](#1-complete-end-to-end-process)
2. [Intake and Triage Process](#2-intake-and-triage-process)
3. [RICE Scoring Decision Tree](#3-rice-scoring-decision-tree)
4. [Kanban Workflow](#4-kanban-workflow)
5. [Stakeholder Communication Flow](#5-stakeholder-communication-flow)
6. [Weekly Backlog Grooming](#6-weekly-backlog-grooming)
7. [Escalation Process](#7-escalation-process)

---

## 1. Complete End-to-End Process

This diagram shows the complete demand management process from request submission to completion.

```mermaid
graph TD
    Start([Stakeholder Submits Request]) --> Intake[1. INTAKE<br/>architecture-requests@company.com<br/>or Jira Service Desk]

    Intake --> Triage[2. INITIAL TRIAGE<br/>Architect reviews<br/>within 3 days]

    Triage --> Valid{Valid<br/>Request?}

    Valid -->|No| Reject[Reject with<br/>Explanation]
    Reject --> NotifyReject[Notify Stakeholder]
    NotifyReject --> End1([End])

    Valid -->|Yes| Scoring[3. RICE SCORING<br/>Calculate:<br/>R × I × C / E]

    Scoring --> Priority[4. ASSIGN PRIORITY<br/>P0: ≥100<br/>P1: 50-99<br/>P2: 20-49<br/>P3: <20]

    Priority --> Backlog[5. ADD TO BACKLOG<br/>Sorted by RICE Score]

    Backlog --> Groom{6. WEEKLY<br/>BACKLOG<br/>GROOMING}

    Groom --> Ready[Move to READY<br/>5-8 items]

    Ready --> WIP{WIP Limit<br/>Available?}

    WIP -->|No| Wait[Wait in READY Queue]
    Wait --> Groom

    WIP -->|Yes| InProgress[7. IN PROGRESS<br/>Max 3 items<br/>Definition of Ready met]

    InProgress --> Work[Execute Work:<br/>Design, Document,<br/>Implement, Test]

    Work --> Review[8. PEER REVIEW<br/>Technical validation<br/>Quality check]

    Review --> QA{Quality<br/>Pass?}

    QA -->|No| Rework[Return to<br/>In Progress]
    Rework --> Work

    QA -->|Yes| DoD{Definition<br/>of Done<br/>Met?}

    DoD -->|No| Complete[Complete<br/>Missing Items]
    Complete --> Review

    DoD -->|Yes| Done[9. DONE<br/>Upload to Confluence<br/>Close Jira ticket]

    Done --> Notify[10. NOTIFY STAKEHOLDER<br/>Share link<br/>Collect feedback]

    Notify --> Metrics[Update Metrics:<br/>Lead Time, Cycle Time,<br/>Throughput]

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

    B -->|Email| C[architecture-requests@<br/>company.com]
    B -->|Jira| D[Jira Service Desk<br/>Request Form]

    C --> E[Auto-create<br/>Jira Ticket]
    D --> E

    E --> F[Assign to<br/>Architecture Team]

    F --> G[Architect Reviews<br/>within 3 days]

    G --> H{Complete<br/>Information?}

    H -->|No| I[Request More Info<br/>from Stakeholder]
    I --> J{Response<br/>Received?}
    J -->|No after 7 days| K[Close as<br/>Incomplete]
    J -->|Yes| G

    H -->|Yes| L{In Scope<br/>for Architecture<br/>Team?}

    L -->|No| M[Redirect to<br/>Appropriate Team]
    M --> N[Notify Stakeholder]

    L -->|Yes| O{Similar Request<br/>Already Exists?}

    O -->|Yes| P[Link to Existing<br/>Request/Documentation]
    P --> Q[Notify Stakeholder]

    O -->|No| R[Proceed to<br/>RICE Scoring]

    R --> S[Gather Scoring<br/>Information:<br/>Reach, Impact,<br/>Confidence, Effort]

    S --> T{Stakeholder<br/>Input Needed?}

    T -->|Yes| U[Schedule<br/>15-min Call]
    U --> V[Confirm Values]

    T -->|No| V
    V --> W[Calculate<br/>RICE Score]

    style A fill:#e1f5ff
    style W fill:#e1ffe1
```

---

## 3. RICE Scoring Decision Tree

How to calculate each component of RICE.

```mermaid
graph TD
    Start([RICE Scoring]) --> Reach{REACH<br/>How many affected?}

    Reach -->|1-10 users/plants| R1[Reach = 5]
    Reach -->|10-25 users/plants| R2[Reach = 15]
    Reach -->|25-50 users/plants| R3[Reach = 35]
    Reach -->|50-100 users/plants| R4[Reach = 75]
    Reach -->|100+ users/plants| R5[Reach = 100]

    R1 --> Impact{IMPACT<br/>How much will it help?}
    R2 --> Impact
    R3 --> Impact
    R4 --> Impact
    R5 --> Impact

    Impact -->|Minimal improvement| I1[Impact = 1]
    Impact -->|Noticeable improvement| I2[Impact = 2]
    Impact -->|Significant improvement| I3[Impact = 3]
    Impact -->|Major improvement| I4[Impact = 4]
    Impact -->|Critical/Transformative| I5[Impact = 5]

    I1 --> Confidence{CONFIDENCE<br/>How certain?}
    I2 --> Confidence
    I3 --> Confidence
    I4 --> Confidence
    I5 --> Confidence

    Confidence -->|Speculative, no validation| C1[Confidence = 50%]
    Confidence -->|Some validation| C2[Confidence = 80%]
    Confidence -->|Validated, clear requirements| C3[Confidence = 100%]

    C1 --> Effort{EFFORT<br/>Person-days needed?}
    C2 --> Effort
    C3 --> Effort

    Effort -->|1-3 days| E1[Effort = 2]
    Effort -->|3-10 days| E2[Effort = 7]
    Effort -->|10-30 days| E3[Effort = 20]
    Effort -->|30-60 days| E4[Effort = 45]
    Effort -->|60+ days| E5[Effort = 80]

    E1 --> Calc[Calculate:<br/>RICE = R × I × C / E]
    E2 --> Calc
    E3 --> Calc
    E4 --> Calc
    E5 --> Calc

    Calc --> Priority{RICE Score}

    Priority -->|≥ 100| P0[P0 - Critical<br/>Start within 2 hours]
    Priority -->|50-99| P1[P1 - High<br/>Start within 3 days]
    Priority -->|20-49| P2[P2 - Medium<br/>Start within 2 weeks]
    Priority -->|< 20| P3[P3 - Low<br/>Backlog, no commitment]

    P0 --> Backlog[Add to Prioritized<br/>Backlog]
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
    A[BACKLOG<br/>20+ items<br/>Prioritized by<br/>RICE Score] -->|Weekly Grooming| B[READY<br/>5-8 items<br/>All info complete<br/>DoR met]

    B -->|Pull when<br/>WIP < 3| C[IN PROGRESS<br/>≤ 3 items<br/>WIP LIMIT<br/>Active work]

    C -->|Work Complete| D[REVIEW<br/>2-3 items<br/>Peer review<br/>QA check]

    D -->|DoD Met| E[DONE<br/>∞ items<br/>Delivered &<br/>Documented]

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
    Start([New Item in Backlog]) --> Sort[Sort by RICE Score<br/>Highest at top]

    Sort --> Groom{Weekly<br/>Backlog<br/>Grooming?}

    Groom -->|Yes| Ready{Meets<br/>Definition<br/>of Ready?}

    Ready -->|No| Clarify[Get missing info<br/>from stakeholder]
    Clarify --> Sort

    Ready -->|Yes| Move[Move to READY<br/>Keep 5-8 items]

    Move --> WIP{WIP Limit<br/>< 3?}

    WIP -->|No| Wait[Wait in READY]
    Wait --> Daily{Daily<br/>Standup}
    Daily --> WIP

    WIP -->|Yes| Pull[Team member<br/>PULLS item to<br/>IN PROGRESS]

    Pull --> Work[Execute Work]

    Work --> Complete{Work<br/>Complete?}

    Complete -->|No| Work

    Complete -->|Yes| PeerReview[REVIEW:<br/>Peer reviews<br/>technical quality]

    PeerReview --> QA{Quality<br/>Check?}

    QA -->|Issues| Fix[Return to<br/>IN PROGRESS]
    Fix --> Work

    QA -->|Pass| DoD{All DoD<br/>Criteria<br/>Met?}

    DoD -->|No| MissingItems[Complete<br/>missing items]
    MissingItems --> PeerReview

    DoD -->|Yes| Done[Move to DONE<br/>Notify stakeholder]

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
    A[Stakeholder:<br/>'This is urgent!'] --> B{Is it truly<br/>urgent?}

    B -->|Production down<br/>Security breach| C[P0 - Critical<br/>Start immediately]

    B -->|Business deadline<br/>Major impact| D[Calculate RICE<br/>Score objectively]

    D --> E{RICE Score?}

    E -->|≥ 100| C
    E -->|50-99| F[P1 - High<br/>Start within 3 days]
    E -->|20-49| G[P2 - Medium<br/>Start within 2 weeks]
    E -->|< 20| H[P3 - Low<br/>In backlog]

    F --> I{Stakeholder<br/>Pushes Back?}
    G --> I
    H --> I

    I -->|Yes| J[Use Communication<br/>Technique]

    J --> K{Which<br/>Technique?}

    K -->|Emotion| L['Let's Get Curious'<br/>Help me understand why...<br/>What happens if...]

    K -->|Comparison| M['Make Constraints Visible'<br/>We have 3 items in progress<br/>Which should we pause?]

    K -->|Options| N['Offer Multiple Options'<br/>A Fast-track + delay X<br/>B Start in 2 weeks<br/>C Reduce scope]

    L --> O[Turn emotion into<br/>problem-solving]
    M --> O
    N --> O

    O --> P{Agreement<br/>Reached?}

    P -->|No| Q[Escalate to<br/>Architecture Lead]

    Q --> R[Architecture Lead<br/>reviews with stakeholder]

    R --> S{Resolution?}

    S -->|No| T[Escalate to<br/>Chief Architect]
    S -->|Yes| U[Document decision<br/>Update priority if needed]

    P -->|Yes| U
    T --> U

    U --> V[Communicate plan<br/>to stakeholder]

    V --> W[Track in Jira<br/>Send regular updates]

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
    Start([Monday 10:00 AM<br/>Backlog Grooming<br/>Meeting]) --> Review[Review all BACKLOG<br/>items]

    Review --> Sort[Sort by<br/>RICE Score]

    Sort --> Check{Each item:<br/>Still relevant?}

    Check -->|No| Archive[Archive item<br/>Notify stakeholder]

    Check -->|Yes| Info{Complete<br/>information?}

    Info -->|No| Request[Request info<br/>from stakeholder]
    Request --> Flag[Flag for<br/>next week]

    Info -->|Yes| Score{RICE Score<br/>still accurate?}

    Score -->|No| Rescore[Recalculate<br/>RICE Score]
    Rescore --> Update[Update priority]

    Score -->|Yes| Ready{Meets<br/>Definition<br/>of Ready?}

    Ready -->|No| Missing[Identify missing<br/>information]
    Missing --> Request

    Ready -->|Yes| Move[Move top 5-8 items<br/>to READY column]

    Update --> Ready
    Archive --> Next
    Flag --> Next

    Move --> Capacity[Review team<br/>capacity this week]

    Capacity --> Plan[Plan which items<br/>move to IN PROGRESS]

    Plan --> Communicate[Send weekly update<br/>to stakeholders:<br/>- What's starting<br/>- What's in progress<br/>- What completed]

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
    A[Stakeholder Disagrees<br/>with Priority] --> B[Architect Explains<br/>RICE Scoring]

    B --> C{Stakeholder<br/>Accepts?}

    C -->|Yes| D[No further<br/>action needed]

    C -->|No| E[Architect offers to<br/>recalculate with<br/>stakeholder input]

    E --> F[Joint session to<br/>review scoring criteria]

    F --> G{New RICE<br/>Score changes<br/>priority?}

    G -->|Yes| H[Update priority<br/>Document rationale]
    H --> I[Communicate<br/>to stakeholder]

    G -->|No| J{Stakeholder still<br/>disagrees?}

    J -->|No| D

    J -->|Yes| K[Escalate to<br/>Architecture Lead]

    K --> L[Architecture Lead<br/>reviews case]

    L --> M{Business<br/>justification<br/>overrides RICE?}

    M -->|Yes| N[Create exception<br/>Document in ADR]
    N --> O[Chief Architect<br/>approval required]

    M -->|No| P[Maintain RICE<br/>priority]

    O --> Q{Approved?}

    Q -->|Yes| H
    Q -->|No| P

    P --> R[Architecture Lead<br/>explains decision<br/>to stakeholder]

    R --> S{Stakeholder<br/>escalates to<br/>executive?}

    S -->|No| T[Close escalation<br/>Maintain priority]

    S -->|Yes| U[Executive review<br/>by CTO/CIO]

    U --> V[Final decision<br/>by executive]

    V --> W{Override<br/>priority?}

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
    A[Demand Management<br/>Metrics] --> B[Lead Time<br/>Request → Done<br/>Target: < 21 days]

    A --> C[Cycle Time<br/>In Progress → Done<br/>Target: < 7 days]

    A --> D[Throughput<br/>Items completed/month<br/>Target: 8-10]

    A --> E[WIP Compliance<br/>% time WIP ≤ 3<br/>Target: > 95%]

    A --> F[Priority Distribution<br/>P0/P1/P2/P3 mix<br/>Target: Balanced]

    A --> G[Stakeholder<br/>Satisfaction<br/>Target: > 8/10]

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
    A[📥 INTAKE<br/>Single entry point] --> B[📊 SCORE<br/>RICE calculation]

    B --> C[🎯 PRIORITIZE<br/>P0 → P3]

    C --> D[📋 BACKLOG<br/>Weekly grooming]

    D --> E[⚙️ EXECUTE<br/>WIP limit: 3]

    E --> F[✅ REVIEW<br/>Peer validation]

    F --> G[✨ DONE<br/>Deliver & notify]

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
- **Parallelogram**: Input/Output

### Colors

- 🔵 **Blue**: Information/Input
- 🟡 **Yellow**: Review/Decision
- 🟢 **Green**: Completion/Success
- 🔴 **Red**: Critical/Urgent
- 🟣 **Purple**: Calculation/Processing

### Arrows

- **Solid**: Primary flow
- **Dashed**: Alternative/Exception flow

---

## How to Use These Diagrams

1. **For Presentations**: Use diagram #1 (Complete End-to-End) or Summary Flow
2. **For Training**: Use all diagrams in sequence
3. **For Reference**: Print diagram #4 (Kanban Workflow) and post near team workspace
4. **For Stakeholders**: Use diagram #5 (Stakeholder Communication Flow)
5. **For Weekly Meetings**: Use diagram #6 (Backlog Grooming)

---

**Document Control:**

- **Created**: 2025-03-21
- **Format**: Mermaid diagrams (compatible with GitHub, Confluence, GitLab, VS Code)
- **Related Documents**: 
  - Demand Management and Prioritization Process
  - Kanban Board Guide
  - Stakeholder Communication Playbook
