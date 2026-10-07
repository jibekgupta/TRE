# A4. Sequence diagrams

Lifelines use the class names from A5. A solid arrow with a filled head is a synchronous call. An open arrowhead is asynchronous. A dashed arrow is a return. Guards are in brackets.

## UC-6 Request dataset access

**Caption:** This sequence supports PP-4 and matches the UC-6 steps, including the rejection and withdrawal paths.

```mermaid
sequenceDiagram
    actor Researcher
    participant IdP as :IdentityProvider
    participant AR as :AccessRequest
    participant DS as :Dataset
    participant AU as :AuditRecord

    Researcher->>IdP: verify(session)
    activate IdP
    IdP-->>Researcher: researcherId
    deactivate IdP

    Researcher->>AR: submit(purpose, benefit, datasetId)
    activate AR
    AR->>DS: isPublished()
    activate DS
    DS-->>AR: published
    deactivate DS

    alt purpose is empty OR published = false
        AR-->>Researcher: rejected
    else purpose is not empty AND published = true
        AR->>AU: record(submitted)
        activate AU
        AU-->>AR: auditId
        deactivate AU
        AR-->>Researcher: requestId
        opt researcher withdraws before review
            Researcher->>AR: withdraw()
            AR->>AU: record(withdrawn)
            activate AU
            AU-->>AR: auditId
            deactivate AU
        end
    end
    deactivate AR
```

![Sequence for UC-6](A4-sequence-uc6.png)

Identity provider is the external system from the context diagram. The alt fragment is the failure path from UC-6. The opt fragment is the cancellation path.

## UC-7 Review access request

**Caption:** This sequence supports PP-4 and matches the UC-7 steps, including a withdrawn request and a denial with no reason.

```mermaid
sequenceDiagram
    actor Custodian
    participant AR as :AccessRequest
    participant AU as :AuditRecord
    participant NS as :NotificationService

    Custodian->>AR: startReview()
    activate AR
    alt status = withdrawn
        AR-->>Custodian: closed
    else decision = deny AND reason is empty
        AR-->>Custodian: rejected
    else decision = approve AND custodian is assigned
        Custodian->>AR: approve()
        AR->>AU: record(approved)
        activate AU
        AU-->>AR: auditId
        deactivate AU
        AR-)NS: notify(researcherId, approved)
        activate NS
        NS-->>AR: accepted
        deactivate NS
    else decision = deny AND reason is not empty
        Custodian->>AR: deny(reason)
        AR->>AU: record(denied)
        activate AU
        AU-->>AR: auditId
        deactivate AU
        AR-)NS: notify(researcherId, denied)
        activate NS
        NS-->>AR: accepted
        deactivate NS
    end
    deactivate AR
```

![Sequence for UC-7](A4-sequence-uc7.png)

Notification service is the external system from the context diagram. The notify messages use an open arrowhead because the TRE does not wait for the message to be delivered. The returns are dashed.
