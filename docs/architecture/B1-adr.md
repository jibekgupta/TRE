# B1. Architecture decision record

**Caption:** This decision supports PP-1, PP-2, PP-3, PP-4, and PP-5. The driving pain point is PP-4, because a custodian decision has to be stored with the same dataset, dictionary, and translation the other pain points create.

## 1. Context

Four people are building the first version. The users are a researcher, a clinical coordinator, and a data custodian. Load is a class-project load: uploads during an active study, and access requests a few times a week, not a national surveillance feed.

The data is sensitive health data. The source file and the dictionary must stay together, because a released analysis file with no local-term definitions would recreate PP-3. A translation must not be published until Dr. Kang accepts it (PP-5). The approval in PP-4 has to be one transaction with the audit record. If the decision were saved and the audit write failed, Daniel Okoro would not be able to answer who received the file.

## 2. Decision

We will build a **modular monolith**. One application, one deployable, one database. Inside it, four modules match the use cases:

| Module | Use cases | What it owns |
|---|---|---|
| Intake | UC-1, UC-2, UC-3 | Upload, check report, local terms |
| Dictionary | UC-4 | Field definitions and the analysis file |
| Translation review | UC-5 | Suggested text and the accepted text |
| Access | UC-6, UC-7 | AccessRequest and AuditRecord |

The modules call each other in process. The Kubernetes manifest in Part C deploys this one application.

## 3. Alternatives considered

**Microservices.** Intake, dictionary, translation review, and access would be four services. We rejected that. The team is four students, and the approval path needs the dataset status, the accepted analysis file, and the audit row at the same time. Splitting them puts a network call and a partial-failure case in the middle of PP-4. We would also operate four images before we have a second real user group.

**Event-driven.** A broker would carry "dataset published" and "request decided" events. We rejected that as the core style. The coordinator and the custodian are waiting on a form. They need a direct answer, not a later event. The one place we still want a non-blocking call is notification, and that is a single outbound call from the access module. A broker for every step would add a component the pain points do not need.

**Serverless.** Each use case would be a function. We rejected that. File checks and dictionary drafts run longer than a short request, and the approval rules would be scattered across functions. A long-running application process is easier for this team to trace from the class diagram to the manifest.

## 4. Performance consequences

Latency on the approval path is the database round trip inside the monolith, plus the identity check at the start of UC-6. There is no service-to-service hop between AccessRequest and Dataset. The slow calls are the ones that leave the process: object storage while Dr. Kamau uploads, and the translation service while Dr. Kang asks for a suggestion. Those calls sit behind UC-1 and UC-5, not behind the custodian's decision.

The application scales by running more copies of the same image. The Part C manifest starts at two replicas and adds pods when CPU stays high. The database is the shared bottleneck. Published status is a small read on every request, so the access module may cache isPublished for a dataset until publish() changes it. Notification is asynchronous, which is the open arrow on the UC-7 sequence. approve() returns after the audit row is stored. It does not wait for email delivery.

## 5. Security consequences

Trust boundaries are the user network, the application, the health-data stores, and the external partners. The picture is B2. Every crossing is HTTPS, except the database, which is SQL over TLS.

Authentication is the identity provider. IdentityProvider.verify(session) is the first call in UC-6. The TRE does not store passwords. Authorization is the User subtype. Coordinator can upload. Researcher can request access and accept a dictionary or a translation. Only Custodian can startReview, approve, deny, and revoke.

Secrets are the database password and the translation API key. They come from a Kubernetes Secret. The manifest points at that Secret and does not contain the values. Source uploads live in object storage. The database holds the dictionary, terms, accepted translations, requests, and audit rows. Approval releases the analysis file only. The source upload stays in storage. That is the PP-4 control.

## 6. Cloud-agnostic plan

| Capability | Open standard | Lock-in we accept |
|---|---|---|
| Run the application | Kubernetes Deployment and Service | None in the manifest. A managed Kubernetes control plane is replaceable with another conformant cluster. |
| Sign-in | OpenID Connect | We accept whichever identity provider speaks OIDC. |
| File storage | S3-compatible API | We do not depend on one vendor's storage features beyond put and get. |
| Records | SQL, PostgreSQL wire protocol | The schema is ordinary SQL. |
| Suggested translation | HTTPS JSON | We accept a translation API. The accepted text is stored in our database, so a provider can be swapped. |
| Notification | HTTPS JSON | Delivery is external. The audit record does not depend on it. |

We do not accept lock-in for the approval rules. Those stay in the access module, which we build.
