# A2. Use case diagram

**Caption:** This use case diagram supports PP-1, PP-2, PP-3, PP-4, and PP-5. Each use case is tagged with the pain point it addresses.

```mermaid
flowchart LR
    subgraph actors [" "]
        direction TB
        coordinator(["Clinical coordinator<br/>Dr. Amina Kamau"])
        researcher(["Researcher<br/>Dr. Ezer Kang"])
        translator(["Translation service<br/>system actor"])
        custodian(["Data custodian<br/>Daniel Okoro"])
        notifier(["Notification service<br/>system actor"])
    end

    subgraph tre ["Global Health TRE"]
        direction TB
        UC1(["UC-1 Upload dataset<br/>PP-2"])
        UC3(["UC-3 Document local term<br/>PP-3"])
        UC2(["UC-2 Check dataset<br/>PP-2"])
        UC4(["UC-4 Prepare analysis file<br/>PP-1"])
        UC5(["UC-5 Translate study text<br/>PP-5"])
        UC6(["UC-6 Request dataset access<br/>PP-4"])
        UC7(["UC-7 Review access request<br/>PP-4"])
    end

    coordinator --- UC1
    coordinator --- UC3
    UC1 -.->|include| UC2
    UC3 -.->|extend| UC1
    researcher --- UC4
    researcher --- UC5
    researcher --- UC6
    translator --- UC5
    custodian --- UC7
    notifier --- UC7

    style actors fill:none,stroke:none
    style tre stroke-dasharray: 6 4
```

![Use case diagram](A2-use-cases.png)

The exported figure is drawn from `A2-use-cases.puml` so the use cases sit inside the system boundary as ovals. The Mermaid block above is the same model: the same actors, the same seven use cases, the same include, and the same extend.

Human actors are the three personas. Translation service and Notification service are system actors. They also appear on the context diagram.

The **include** arrow points from UC-1 Upload dataset to UC-2 Check dataset. Every upload runs the check. The **extend** arrow points from UC-3 Document local term back to UC-1 Upload dataset. Documenting a term is extra behavior, used when the file contains a local term, and it is not required for every upload.
