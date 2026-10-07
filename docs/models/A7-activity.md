# A7. Activity diagram

**Caption:** This activity diagram supports PP-4, the pain point the walkthrough traces: a researcher asks for a published dataset, and the custodian decision is recorded before any analysis file is released.

```mermaid
flowchart TB
    subgraph researcherLane ["Swimlane: Researcher - Dr. Ezer Kang"]
        startNode([Start]) --> openAct[open] --> enterAct[Enter purpose and benefit] --> submitAct[submit]
    end

    subgraph treLane ["Swimlane: TRE"]
        purposeGate{purpose not empty?}
        publishedGate{dataset published?}
        rejectAct[Reject request]
        rejectEnd([Stop])
        recordSubmitted[record submitted]
        withdrawnGate{status withdrawn?}
        closedAct[Return closed]
        closedEnd([Stop])
        reasonGate{deny reason not empty?}
        decisionFork{{fork}}
        recordDecision[record decision]
        decisionJoin{{join}}
        releaseAct[Release analysis file]
        revokeGate{revoke later?}
        doneNode([End])
    end

    subgraph custodianLane ["Swimlane: Data custodian - Daniel Okoro"]
        reviewAct[startReview]
        decisionGate{approve or deny?}
        approveAct[approve]
        denyAct[deny]
        revokeAct[revoke]
    end

    subgraph notifyLane ["Swimlane: Notification service"]
        notifyAct[notify researcher]
    end

    submitAct --> purposeGate
    purposeGate -->|no| rejectAct --> rejectEnd
    purposeGate -->|yes| publishedGate
    publishedGate -->|no| rejectAct
    publishedGate -->|yes| recordSubmitted --> reviewAct
    reviewAct --> withdrawnGate
    withdrawnGate -->|yes| closedAct --> closedEnd
    withdrawnGate -->|no| decisionGate
    decisionGate -->|approve| approveAct --> decisionFork
    decisionGate -->|deny| reasonGate
    reasonGate -->|no| decisionGate
    reasonGate -->|yes| denyAct --> decisionFork
    decisionFork --> recordDecision --> decisionJoin
    decisionFork --> notifyAct --> decisionJoin
    decisionJoin --> releaseAct --> revokeGate
    revokeGate -->|no| doneNode
    revokeGate -->|yes| revokeAct --> doneNode
```

![Activity diagram](A7-activity.png)

The swimlanes are the researcher persona, the TRE, the custodian persona, and the notification service from the context diagram.

The decision diamonds are guarded. An empty purpose or an unpublished dataset stops the flow. A denial with an empty reason returns to the custodian. Approve and deny then fork. Recording the audit decision and notifying the researcher run in parallel, then join. The analysis file is released only after that join. Revoke is the later path from a released file back to the custodian, matching the Approved to Revoked transition.
