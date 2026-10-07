# B2. Architecture diagram

**Caption:** This architecture supports PP-1, PP-2, PP-3, PP-4, and PP-5. The TRE application is the modular monolith from B1. Arrows that cross a trust boundary are labeled with the protocol.

Legend, matching the lecture: a box is a client, a service, or an external system. A cylinder is a data store. A dashed box is a trust boundary. A solid arrow is synchronous. A dotted arrow is asynchronous. There is no message broker. B1 rejected an event-driven core. Notification is a direct async call.

```mermaid
flowchart TB
    subgraph users ["Trust boundary: user network"]
        researcher["Researcher client<br/>Dr. Ezer Kang"]
        coordinator["Coordinator client<br/>Dr. Amina Kamau"]
        custodian["Custodian client<br/>Daniel Okoro"]
    end

    subgraph appZone ["Trust boundary: TRE application"]
        app["TRE application<br/>Intake, Dictionary,<br/>Translation review, Access"]
    end

    subgraph dataZone ["Trust boundary: health data"]
        db[("PostgreSQL")]
        files[("Object storage")]
    end

    subgraph partners ["Trust boundary: external partners"]
        idp["Identity provider"]
        mt["Translation service"]
        ns["Notification service"]
    end

    researcher -->|HTTPS sync| app
    coordinator -->|HTTPS sync| app
    custodian -->|HTTPS sync| app
    app -->|SQL/TLS sync| db
    app -->|HTTPS sync| files
    app -->|HTTPS sync OIDC| idp
    app -->|HTTPS sync| mt
    app -.->|HTTPS async| ns
```

![Architecture diagram](B2-architecture.png)

The modules inside the application match the class clusters. Intake and Dictionary own Dataset, FieldDefinition, and LocalTerm. Translation review owns Translation. Access owns AccessRequest and AuditRecord. Identity provider and Notification service are the same external systems as on the context diagram and the class diagram.

Each arrow that leaves a dashed box is labeled. Authentication at the identity provider is the verify(session) call in the UC-6 sequence. The security story for these crossings is in B1 section 5.
