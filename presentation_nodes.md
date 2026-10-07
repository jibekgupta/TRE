# Presentation notes

**Product:** Global Health Trusted Research Environment (TRE)
**Team:** Anu, Ade, Manushi, Jibek
**Thread:** PP-4. Daniel Okoro will release an analysis file only when AccessRequest stores the requester, the purpose, the expected benefit, the decision, and the time.

Use these names in every section: `AccessRequest`, `Dataset`, `AuditRecord`, `UC-6`, `UC-7`, `PP-4`.

The short timing is the live 10-minute walk. The text under each figure is the full technical explanation of every element. If the clock is tight, say the bold line, point at the element, and move.

| Time | Speaker | Figures |
|---|---|---|
| 0:00-2:15 | Anu | Scope, A1 context, A2 use cases |
| 2:15-4:30 | Ade | A3 textual use cases, A4 sequences |
| 4:30-7:00 | Manushi | A5 class diagram, A6 state machine |
| 7:00-10:00 | Jibek | A7 activity, B2 architecture, B1 decision, `deploy/tre-app.yaml` |

## Shared notation

| Symbol | Meaning in our figures |
|---|---|
| Dashed box | A boundary. On A1 it is the system boundary. On B2 it is a trust boundary. |
| Oval | A use case. Named verb + noun, tagged with a PP-ID. |
| Stick figure | An actor. `<<system>>` means the actor is software, not a persona. |
| Solid line, no arrowhead | An association. The actor participates in that use case. |
| Dashed arrow `«include»` | The base use case always runs the included one. Arrow points at the included use case. |
| Dashed arrow `«extend»` | Optional behavior. Arrow points from the extension back to the base use case. |
| Lifeline `:ClassName` | One object. The colon means "an instance of this class." |
| Filled arrowhead, solid line | Synchronous call. The sender waits. |
| Open arrowhead | Asynchronous call. The sender does not wait. Used for `notify`. |
| Dashed arrow back to the caller | A return value. |
| Thin bar on a lifeline | Activation. The object is executing that call. |
| `alt` | Combined fragment. The branches are mutually exclusive. Guards are in brackets. |
| `opt` | Combined fragment. The block runs only when its guard is true. |
| Hollow triangle on a class | Generalization. The triangle sits on the parent. |
| Filled diamond | Composition. The part dies with the whole. Diamond sits on the whole. |
| Hollow diamond | Aggregation. The part can outlive the whole. Diamond sits on the whole. |
| `1`, `0..*`, `1..*` | Multiplicity. How many objects sit on that end of the association. |
| Black dot | Initial pseudostate. |
| Bullseye | Final state. |
| `event [guard] / action` | State transition label. Guard must be true. Action runs as the transition fires. |
| Diamond in an activity | A decision. Outgoing edges are guards. |
| Bar labeled fork, bar labeled join | Parallel split and parallel join. |
| Cylinder | A data store. |
| Solid architecture arrow | Synchronous protocol call. |
| Dotted architecture arrow | Asynchronous protocol call. |

---

## Anu

**Files:** `docs/scope.md`, `docs/models/A1-context.md`, `docs/models/A2-use-cases.md`, `docs/models/A2-use-cases.puml`

### What you say first

"The walkthrough entity is an AccessRequest. PP-4 is Daniel Okoro's pain: sensitive health data can leave an institution with no stored record of the requester, the purpose, and the benefit to the contributing population. UC-6 creates that record. UC-7 stores his decision. Approval releases `analysisFileName` only. The source object stays in object storage."

### Scope table, row by row

Point at the traceability table. Every later ID is defined here.

| ID | Persona | Technical capability | Use cases | Success signal you can cite |
|---|---|---|---|---|
| PP-1 | Dr. Ezer Kang | `Dataset.prepareAnalysisFile()` drafts a dictionary row per column (`fieldName`, `label`, `dataType`) and a rectangular file. `acceptAnalysisFile()` commits it. | UC-4 | Every uploaded column has one `FieldDefinition`. |
| PP-2 | Dr. Amina Kamau | `Coordinator.upload(fileName)` accepts a CSV or spreadsheet as-is. `Dataset.check()` reports missing values and field shape. | UC-1, UC-2 | One sitting, under 15 minutes of her work. |
| PP-3 | Dr. Amina Kamau | `LocalTerm.define(text)` is required before publish when she flags a term. The definition travels with the released file. | UC-3 | A flagged term cannot be published with an empty definition. |
| PP-4 | Daniel Okoro | `AccessRequest.submit(purpose, benefit, datasetId)` refuses an empty purpose. `approve()` / `deny(reason)` persist custodian identity and `decidedAt`. | UC-6, UC-7 | He can list requester, purpose, benefit, decision, and timestamp. |
| PP-5 | Dr. Ezer Kang | `Translation.suggest()` returns machine text. `Translation.accept(text)` is the only path that sets `acceptedText`. Unpublished suggestions stay off the catalog. | UC-5 | Source text, accepted text, language, and acceptor are stored together. |

PP-1, PP-2, and PP-3 are what make a Dataset publishable before UC-6 can succeed. PP-5 is in the model and is off today's path.

Out of scope, if asked: catalog search, in-TRE notebooks, Likert redesign, back-translation as a method, qualitative coding, a hospital EHR, and any release that skips `AccessRequest`.

### Figure A1, context diagram

The figure has one dashed boundary and eight boxes. Lines are undirected. This diagram shows dependency, not message order. Message order is Ade's sequence.

**Inside the dashed boundary**

- `Global Health TRE`. One system. We build intake, the check report, the data dictionary, local-term definitions, translation review, and the approval record inside this box. It is one process, which is why B2 later shows one application.

**Above the boundary, human actors**

- `Researcher / Dr. Ezer Kang`. Calls UC-4, UC-5, and UC-6. In PP-4 he is the subject of the access record.
- `Clinical coordinator / Dr. Amina Kamau`. Calls UC-1 and UC-3. She produces the Dataset that UC-6 names.
- `Data custodian / Daniel Okoro`. Calls UC-7. He is the only actor who can `approve`, `deny`, or `revoke`.

**Below the boundary, external systems**

- `Identity provider`. Owns accounts and passwords. The TRE calls `verify(session)` and stores no password. Protocol on B2 is HTTPS with OpenID Connect.
- `Object storage`. Holds the source upload. S3-compatible put/get. The analysis file is a separate object. Approval does not copy the source object out.
- `Translation service`. Owns the language model. It returns `suggestedText`. The TRE persists text only after `accept(text)`.
- `Notification service`. Delivers the decision to the researcher. The call is asynchronous. The audit row does not depend on delivery.

**What the lines mean.** Each line is "this actor or system participates with the TRE." There is no arrow because a context diagram does not specify who sends the first message. The boundary justification is the build-versus-integrate split: the approval record is inside; identity and machine translation are outside.

### Figure A2, use case diagram

Show the exported PNG (`A2-use-cases.png`). Ovals are inside the rectangle `Global Health TRE`. Stick figures are outside. That rectangle is the same boundary as A1.

**Actors and the ovals they touch**

| Actor | Line goes to | Why |
|---|---|---|
| Clinical coordinator, Dr. Amina Kamau | UC-1 Upload dataset (PP-2), UC-3 Document local term (PP-3) | She contributes the file and the local definitions. |
| Researcher, Dr. Ezer Kang | UC-4 Prepare analysis file (PP-1), UC-5 Translate study text (PP-5), UC-6 Request dataset access (PP-4) | He accepts the file and the translation, then requests release. |
| Translation service `<<system>>` | UC-5 Translate study text (PP-5) | System actor. It supplies `suggest()`, and it is the translation box on A1. |
| Data custodian, Daniel Okoro | UC-7 Review access request (PP-4) | He is the authorization decision. |
| Notification service `<<system>>` | UC-7 Review access request (PP-4) | System actor. It receives `notify` after the decision, and it is the notification box on A1. |

**Every oval**

- **UC-1 Upload dataset, PP-2.** Base use case. `Coordinator.upload(fileName)`. Stores a `Dataset`.
- **UC-2 Check dataset, PP-2.** Included by UC-1. `Dataset.check()` always runs. The dashed `«include»` arrow points from UC-1 to UC-2. Direction matters: the including use case points at the included one.
- **UC-3 Document local term, PP-3.** Extends UC-1. `LocalTerm.define(text)`. The dashed `«extend»` arrow points from UC-3 back to UC-1. Direction matters: the extension points at the base. The behavior runs only when the file contains a local term. An upload with no local term does not take this path.
- **UC-4 Prepare analysis file, PP-1.** `prepareAnalysisFile()` then `acceptAnalysisFile()`. Produces the object that an approved request is allowed to release.
- **UC-5 Translate study text, PP-5.** `suggest()` then a required `accept(text)`. Unreviewed machine text is not published.
- **UC-6 Request dataset access, PP-4.** Today's first detailed use case. `submit(purpose, benefit, datasetId)`.
- **UC-7 Review access request, PP-4.** Today's second detailed use case. `startReview()`, then `approve()` or `deny(reason)`.

Seven use cases, which is inside the required 6 to 10. Two of the five actors are system actors. Both system actors also appear on A1, which is the consistency rule.

**Handoff:** "Ade has the preconditions, the stimulus, and the exact calls for UC-6 and UC-7."

**If asked**

- Include is mandatory and points forward. Extend is conditional and points backward at the base.
- A persona who is not on A2 is not in the MVP scope. Marcus Lee and Bayowa O. are not actors on this diagram.
- UC-6 does not download the source file. The response is a `requestId` and status `Submitted`.

---

## Ade

**Files:** `docs/models/A3-textual-use-cases.md`, `docs/models/A4-sequences.md`

### Figure A3, UC-6 Request dataset access

Read the table as a contract. The sequence must implement every row.

| Row | Technical content |
|---|---|
| Actors | Researcher (Dr. Ezer Kang). Identity provider. Inside the TRE: `AccessRequest`, `Dataset`, `AuditRecord`. |
| Description | He requests the accepted analysis file for one published Dataset. Arguments are purpose and benefit. |
| Data | `session`, `datasetId`, `purpose`, `benefit`. |
| Stimulus | Submit on the access-request form. That stimulus is the message `submit(purpose, benefit, datasetId)`. |
| Response | Return `requestId`. `AccessRequest.status = Submitted`. `AuditRecord.action = submitted`. |
| Precondition | A session exists, and the Dataset row exists. `verify(session)` establishes the first. `datasetId` names the second. |
| Postcondition | Happy path: one `AccessRequest` in `Submitted`, with purpose and benefit non-empty. Alternative path: no `Submitted` row is kept. |
| Comment | Source object is not copied to the researcher here. Release happens only after UC-7 `approve()`. |
| Alternative 1 | Guard: `purpose` is empty OR `Dataset.isPublished() = false`. Return `rejected`. No `record(submitted)`. |
| Alternative 2 | Before review, `withdraw()`. `AuditRecord.record(withdrawn)`. Status becomes `Withdrawn`. |

### Figure A4, sequence UC-6

Lifelines left to right: `Researcher`, `:IdentityProvider`, `:AccessRequest`, `:Dataset`, `:AuditRecord`. The colon is UML instance notation. Activation bars show who is on the stack.

Walk the messages in order:

1. `Researcher -> :IdentityProvider : verify(session)`. Synchronous, filled head. The researcher waits. This is the external system from A1. Return, dashed: `researcherId`. Then the identity activation ends. The TRE now has a subject id for the audit record and still has no password.
2. `Researcher -> :AccessRequest : submit(purpose, benefit, datasetId)`. Synchronous. `AccessRequest` activation starts and stays up for the whole fragment, because this object owns the transaction.
3. `:AccessRequest -> :Dataset : isPublished()`. Synchronous. Return, dashed: `published` (Boolean). Dataset activation ends on that return.
4. **`alt` fragment.** Two branches, one runs.
   - Guard `[purpose is empty OR published = false]`. Return `rejected` to the researcher. No insert into `AuditRecord`. This is alternative flow 1 from A3.
   - Guard `[purpose is not empty AND published = true]`.
     1. `:AccessRequest -> :AuditRecord : record(submitted)`. Synchronous. Return `auditId`. The argument `submitted` is the `event` parameter of `record(event)`.
     2. Return `requestId` to the researcher. Status is now `Submitted`. This matches the A3 response row.
     3. **`opt` nested inside this branch.** Guard `[researcher withdraws before review]`. It runs only if he cancels before `startReview`.
        - `Researcher -> :AccessRequest : withdraw()`.
        - `:AccessRequest -> :AuditRecord : record(withdrawn)`. Return a second `auditId`. Status becomes `Withdrawn`. This is alternative flow 2. A withdrawn request is a final state on Manushi's machine, and UC-7 must refuse to decide it.

`AccessRequest` deactivates after the fragment. One activation covers submit, the published check, the audit write, and the optional withdraw, because those steps are one application transaction.

### Figure A3, UC-7 Review access request

| Row | Technical content |
|---|---|
| Actors | Custodian (Daniel Okoro). `AccessRequest`, `AuditRecord`. Notification service. |
| Description | He sets the decision. `approve()` releases the analysis file. The source object remains in object storage. |
| Data | `requestId`, `decision`, deny `reason`, custodian identity, `decidedAt`. |
| Stimulus | He opens a submitted request and chooses approve or deny. The first message is `startReview()`. |
| Response | Status becomes `Approved` or `Denied`. `AuditRecord` stores the decision, the actor, and the time. `NotificationService.notify(userId, message)` informs the researcher. |
| Precondition | Custodian session exists. Request status is `Submitted`. |
| Postcondition | Decision, actor, and `decidedAt` are durable. Researcher has been handed to the notifier. |
| Comment | Authorization rule: only the `Custodian` subtype may call `startReview`, `approve`, `deny`, and `revoke`. |
| Alternative 1 | Status is already `Withdrawn`. Return `closed`. No decision row. |
| Alternative 2 | Decision is deny and `reason` is empty. Return `rejected`. Status stays `InReview`. |

### Figure A4, sequence UC-7

Lifelines: `Custodian`, `:AccessRequest`, `:AuditRecord`, `:NotificationService`.

1. `Custodian -> :AccessRequest : startReview()`. Synchronous. Activation on `AccessRequest` covers the whole `alt`. This moves the state machine from `Submitted` to `InReview`.
2. **`alt` has four mutually exclusive guards.**
   - `[status = withdrawn]`. Dashed return `closed`. No audit write, no notify. Matches alternative 1.
   - `[decision = deny AND reason is empty]`. Dashed return `rejected`. Status stays `InReview`. Matches alternative 2. The guard is why the state machine's deny transition also requires `[reason not empty]`.
   - `[decision = approve AND custodian is assigned]`.
     1. `Custodian -> :AccessRequest : approve()`.
     2. `:AccessRequest -> :AuditRecord : record(approved)`. Return `auditId`. `decidedAt` is set with this write.
     3. `:AccessRequest -) :NotificationService : notify(researcherId, approved)`. Open arrowhead. Asynchronous. The access transaction commits before delivery. Return `accepted` means the notifier took the job, not that the researcher has read it.
   - `[decision = deny AND reason is not empty]`.
     1. `deny(reason)`.
     2. `record(denied)`. Return `auditId`.
     3. `notify(researcherId, denied)`, same open arrow.

The external system on this diagram is `:NotificationService`, the same box as on A1 and B2.

**Handoff:** "Every message I named is an operation on the receiving class. `record(submitted)` and `record(approved)` are one operation, `AuditRecord.record(event)`, with a different argument. Manushi has those operations and the states they move."

**If asked**

- Filled head = synchronous. Open head = asynchronous. Dashed shaft = return.
- `alt` chooses one branch. `opt` may run or be skipped. The `opt` is nested in the success branch because you cannot withdraw a request that was never submitted.
- `notify` is not on the critical path of the audit insert. If the notifier is down, Okoro still has the `AuditRecord`.

---

## Manushi

**Files:** `docs/models/A5-classes.md`, `docs/models/A6-state.md`

### Figure A5, class diagram

Twelve classes. Walk attributes, then operations, then the lines.

**Generalization.** `User <|-- Researcher`, `User <|-- Coordinator`, `User <|-- Custodian`. The hollow triangle is on `User`. Subclasses inherit `userId: UUID`, `name: String`, `email: String`. A person has one role in this MVP. Authorization is the subtype, not a separate permission table.

**`User`**

- `userId: UUID` primary identity inside the TRE, distinct from the identity-provider subject until `verify` maps them.
- `name: String`, `email: String`.

**`Researcher`** extends User. Operation `requestAccess(): UUID` is the domain name for starting UC-6. The note on this class says sign-in is `IdentityProvider.verify(session)`, which is message 1 of the UC-6 sequence. There is no association line drawn to `IdentityProvider`; the note plus the sequence are the link, so the line is not mistaken for Custodian.

**`Coordinator`** extends User. `upload(fileName): UUID` implements UC-1 and returns the new `datasetId`.

**`Custodian`** extends User. `review(requestId): void` is the entry to UC-7. Only this subtype reaches `startReview`, `approve`, `deny`, and `revoke`.

**`Dataset`**

- `datasetId: UUID`, `title: String`, `status: String`, `originNote: String`, `analysisFileName: String`.
- `isPublished(): Boolean` is the call in UC-6.
- `check(): Boolean` is UC-2.
- `publish(): void` is allowed only after the check passes and every flagged `LocalTerm` has a definition.
- `prepareAnalysisFile(): String` and `acceptAnalysisFile(): void` are UC-4. The string is the stored file name.
- `originNote` is the provenance text a researcher reads before requesting access.

**`FieldDefinition`** (composition, PP-1)

- `fieldName: String`, `label: String`, `dataType: String`.
- `accept(): void` commits one dictionary row.
- Multiplicity `Dataset "1" *-- "1..*" FieldDefinition`, label `dictionary`. Filled diamond on `Dataset`. A column definition has no meaning if the dataset is deleted. `1..*` because a published dataset has at least one column.

**`LocalTerm`** (aggregation, PP-3)

- `term: String`, `definition: String`.
- `define(text): void` writes the definition.
- Multiplicity `Dataset "1" o-- "0..*" LocalTerm`, label `aggregation`. Hollow diamond on `Dataset`. `0..*` because a file may have no local terms. The term can be reused on a later dataset from the same site, so deleting one dataset does not delete the term. That is why this one is aggregation and the dictionary is composition.

**`Translation`** (composition, PP-5)

- `sourceText: String`, `suggestedText: String`, `acceptedText: String`, `language: String`.
- `suggest(): String` calls the external translation service and fills `suggestedText`.
- `accept(text): void` copies the researcher's edited text into `acceptedText`. Publish reads `acceptedText` only.
- Multiplicity `Dataset "1" *-- "0..*" Translation`, label `wording`. Filled diamond on `Dataset`. `0..*` because a dataset may have no translated labels. The accepted wording belongs to that dataset's fields.

**`AccessRequest`** (PP-4, the core entity)

- `requestId: UUID`, `purpose: String`, `benefit: String`, `status: String`, `decidedAt: DateTime`.
- `open(): UUID` creates `Draft`. This event is on the activity diagram.
- `submit(purpose, benefit, datasetId): UUID` is the UC-6 call. Returns `requestId`.
- `startReview(): void`, `approve(): void`, `deny(reason): void` are the UC-7 calls.
- `withdraw(): void` is the UC-6 `opt`.
- `revoke(): void` is the later path on the activity diagram, from `Approved`.

**`AuditRecord`**

- `recordId: UUID`, `action: String`, `at: DateTime`.
- `record(event): UUID`. The sequence arguments `submitted`, `withdrawn`, `approved`, and `denied` are four calls to this one operation. `event` is stored in `action`. `at` is the timestamp Okoro needs for PP-4.
- Multiplicity `AccessRequest "1" *-- "1..*" AuditRecord`, label `history`. Filled diamond on `AccessRequest`. `1..*` because a submitted request has at least the `submitted` record. The history is part of the request and is deleted with it.

**`IdentityProvider`**

- `verify(session): UUID`. External system from A1. Returns the researcher id. No password attribute exists on any TRE class.

**`NotificationService`**

- `notify(userId, message): void`. Arguments in the sequence are `(researcherId, approved)` and `(researcherId, denied)`.
- Association `AccessRequest "0..*" --> "1" NotificationService`, label `alertsThrough`. Many requests use the one notifier. The arrow is a reference, not a composition. The notifier outlives every request.

**Other associations and their multiplicities**

- `Coordinator "1" --> "0..*" Dataset`, label `uploads`. One coordinator, many datasets, including zero before the first upload.
- `Researcher "1" --> "0..*" AccessRequest`, label `submits`. One researcher owns many requests.
- `Custodian "1" --> "0..*" AccessRequest`, label `reviews`. One custodian works a queue. Zero means an empty queue.
- `AccessRequest "0..*" --> "1" Dataset`, label `targets`. Many requests may name one dataset. A request names exactly one dataset. Zero requests on a dataset is allowed; the dataset can exist before anyone asks.

### Figure A6, state machine of AccessRequest

One region. Seven states. One initial pseudostate. Three transitions into a final pseudostate.

| From | Event | Guard | Action | To |
|---|---|---|---|---|
| initial `[*]` | `open` | none | none | `Draft` |
| `Draft` | `submit` | `[purpose not empty AND dataset published]` | `/ record submitted` | `Submitted` |
| `Submitted` | `withdraw` | none | `/ record withdrawn` | `Withdrawn` |
| `Submitted` | `startReview` | none | none | `InReview` |
| `InReview` | `approve` | `[custodian assigned]` | `/ notify researcher` | `Approved` |
| `InReview` | `deny` | `[reason not empty]` | `/ record denied` | `Denied` |
| `Approved` | `revoke` | none | `/ notify researcher` | `Revoked` |
| `Denied` | completion |  |  | final `[*]` |
| `Withdrawn` | completion |  |  | final `[*]` |
| `Revoked` | completion |  |  | final `[*]` |

How to say the guards:

- `submit` will not leave `Draft` when purpose is empty or `isPublished()` is false. That is the `alt` failure branch in UC-6. The action `/ record submitted` is `AuditRecord.record(submitted)`.
- `approve` will not fire without an assigned custodian. The action notifies the researcher, which is the open arrow in UC-7.
- `deny` will not fire with an empty reason. The action writes `record(denied)`. The UC-7 branch that returns `rejected` is this guard failing, so the object stays in `InReview`.
- `withdraw` is the cancellation path. It is legal from `Submitted`, which is before review. It is the `opt` fragment.
- `deny` is the failure path.
- `revoke` is a later event from `Approved`. It does not delete the audit history. It moves the request to `Revoked` and notifies the researcher. The analysis file is no longer released.
- `Approved` is not final, because revoke can still happen. `Denied`, `Withdrawn`, and `Revoked` are final.

**Event trace, so the models agree**

| Event | Also appears in |
|---|---|
| `open` | A7 researcher swimlane |
| `submit` | A4 UC-6 and A7 |
| `withdraw` | A4 UC-6 `opt` |
| `startReview` | A4 UC-7 and A7 custodian swimlane |
| `approve` | A4 UC-7 and A7 |
| `deny` | A4 UC-7 and A7 |
| `revoke` | A7 custodian swimlane, after release |

**Handoff:** "Jibek draws these same events as activities in swimlanes, then shows the process and the protocols that host them."

**If asked**

- Composition versus aggregation: dictionary rows, translations, and audit rows are parts. Local terms are shared. The diamond kind is the difference, and it sits on the whole.
- `IdentityProvider` has no multiplicity line. The operation is `verify(session)`, and the UC-6 lifeline is the evidence.
- Status values on `AccessRequest.status` are exactly these state names: `Draft`, `Submitted`, `InReview`, `Approved`, `Denied`, `Withdrawn`, `Revoked`.

---

## Jibek

**Files:** `docs/models/A7-activity.md`, `docs/architecture/B2-architecture.md`, `docs/architecture/B1-adr.md`, `deploy/tre-app.yaml`, `deploy/kubeconform.png`

### Figure A7, activity diagram

Four swimlanes. Each lane is an actor from A1 or the system inside the A1 boundary.

| Swimlane | Participant | Maps to |
|---|---|---|
| Researcher, Dr. Ezer Kang | Persona | Researcher on A1 and A2 |
| TRE | Our system | The single box inside the A1 boundary |
| Data custodian, Daniel Okoro | Persona | Custodian on A1 and A2 |
| Notification service | External partner | Notification box on A1, B2, and the UC-7 lifeline |

**Researcher lane, in order**

- Start node. The activity begins.
- `open`. Creates `AccessRequest` in `Draft`. Same event as the initial transition on A6.
- `Enter purpose and benefit`. Sets the two required strings.
- `submit`. Crosses into the TRE lane. Same message as UC-6.

**TRE lane, decisions first**

- Decision `purpose not empty?`
  - Guard `no` goes to `Reject request`, then a flow-final Stop. No `Submitted` row. Same failure as the UC-6 `alt`.
  - Guard `yes` goes to the next decision.
- Decision `dataset published?`
  - Guard `no` joins the same `Reject request` node.
  - Guard `yes` goes to `record submitted`. That action is `AuditRecord.record(submitted)`. State is now `Submitted`.
- Control passes to the custodian lane at `startReview`.
- Decision `status withdrawn?` comes back into the TRE after `startReview`.
  - Guard `yes`: `Return closed`, then Stop. This is the UC-7 guard `[status = withdrawn]`.
  - Guard `no`: the custodian decision diamond.
- Decision `deny reason not empty?` is reached only on the deny edge.
  - Guard `no`: edge returns to `approve or deny?`. The custodian must enter a reason. Status stays `InReview`.
  - Guard `yes`: action `deny`, then the fork.
- **Fork.** One token splits into two tokens.
  - Token 1, TRE lane: `record decision`. This is `record(approved)` or `record(denied)`.
  - Token 2, notification lane: `notify researcher`. This is the async `notify` call.
- **Join.** The release waits for both tokens. `Release analysis file` runs only after the audit write and the notify handoff. The object released is `Dataset.analysisFileName`. The source upload is not the object on this action.
- Decision `revoke later?`
  - Guard `no`: End node.
  - Guard `yes`: crosses to the custodian action `revoke`, then End. That is the A6 transition `Approved -> Revoked` with action `/ notify researcher`.

**Custodian lane**

- `startReview`. Enters from `record submitted`.
- Decision `approve or deny?`
  - Edge `approve` goes to action `approve`, then the fork. Guard on the state machine is `[custodian assigned]`.
  - Edge `deny` goes to the reason decision, not straight to the fork.
- `revoke` is the only custodian action after release.

**Notification lane**

- One action, `notify researcher`. It sits on the second fork branch so it is parallel with `record decision`, then it enters the join. The lane exists because the notification service is an external actor, and a swimlane is required for each participant.

### Figure B2, architecture diagram

Legend, say it once: box = client, service, or external system. Cylinder = data store. Dashed box = trust boundary. Solid arrow = synchronous. Dotted arrow = asynchronous. No broker. B1 rejected an event-driven core, so there is no queue component to draw.

**Trust boundary: user network**

- `Researcher client`, Dr. Ezer Kang.
- `Coordinator client`, Dr. Amina Kamau.
- `Custodian client`, Daniel Okoro.

These are the three personas' browsers or apps. They are outside the application boundary. Each has a solid arrow into the application labeled `HTTPS sync`. The caller waits for the HTTP response. TLS terminates at the application edge. This is the same population as the actors on A1.

**Trust boundary: TRE application**

- One box: `TRE application`, modules `Intake`, `Dictionary`, `Translation review`, `Access`.
- Module to class cluster:
  - Intake owns `Dataset` upload, `check()`, and `LocalTerm`.
  - Dictionary owns `FieldDefinition`, `prepareAnalysisFile()`, `acceptAnalysisFile()`.
  - Translation review owns `Translation.suggest()` and `Translation.accept()`.
  - Access owns `AccessRequest` and `AuditRecord`, which is UC-6 and UC-7.
- Calls between modules are in-process. They do not cross a trust boundary and they have no protocol label. That is the modular monolith.

**Trust boundary: health data**

- Cylinder `PostgreSQL`. Arrow from the app labeled `SQL/TLS sync`. The database holds dictionary rows, terms, accepted translations, requests, and audit rows. TLS wraps the SQL connection. Synchronous because `approve()` must see the commit.
- Cylinder `Object storage`. Arrow labeled `HTTPS sync`. Source uploads and the analysis file. S3-compatible API. Separate from PostgreSQL so large files are not bytea in SQL.

**Trust boundary: external partners**

- `Identity provider`. Arrow `HTTPS sync OIDC`. This is `verify(session)` in UC-6. OpenID Connect. Synchronous because submit cannot continue without `researcherId`.
- `Translation service`. Arrow `HTTPS sync`. This is `Translation.suggest()`. Synchronous because the researcher is waiting on suggested text in UC-5. It is off the PP-4 critical path.
- `Notification service`. Dotted arrow `HTTPS async`. This is `notify` in UC-7. Dotted because the sequence uses an open arrowhead. The audit commit does not wait for SMTP or push delivery.

Every arrow that leaves a dashed box has a protocol. That is the B2 grading rule. The authentication story for the OIDC arrow and the authorization story for the custodian are in B1 section 5.

### B1, the decision, technical points

Say this after the picture, in this order.

1. **Context.** Four developers. Request rate is weekly, not a surveillance stream. The data is health data. PP-4 requires the decision and the audit insert in one transaction. A saved decision with a failed audit write would leave Okoro unable to answer who received the file.
2. **Decision.** Modular monolith. One deployable, one database, four modules. Kubernetes runs that one image.
3. **Rejected: microservices.** Four network services would split `isPublished()`, the analysis-file name, and `record(event)` across process boundaries. PP-4 would then need a distributed transaction. The team would also run four images.
4. **Rejected: event-driven.** Okoro's form needs a direct response. A broker for every check adds a component no pain point requires. The single non-blocking call is `notify`, and it is one outbound HTTPS call, not a bus.
5. **Rejected: serverless.** `check()` and `prepareAnalysisFile()` are longer than a short function, and the approve/deny guards would be split across functions. One process keeps the class diagram and the manifest aligned.
6. **Performance.** Approval latency is one SQL round trip plus the earlier OIDC check. No service-to-service hop on that path. Slow calls are object storage on upload and the translation API on UC-5. `isPublished()` may be cached until `publish()` invalidates it. Scale is more replicas of the same image. The database is the shared bottleneck. `approve()` returns after the audit commit.
7. **Security.** Authentication is OIDC, not a local password store. Authorization is the `User` subtype. Secrets are Kubernetes Secrets `tre-db` and `tre-api`. The manifest stores the key reference, not the secret bytes. Source objects stay in the health-data boundary. Release copies the analysis file only.
8. **Portability.** Kubernetes for compute, OIDC for sign-in, S3 API for objects, PostgreSQL wire protocol for records, HTTPS JSON for translation and notification. Approval rules stay in the access module we own.

### Figure C, the manifest `deploy/tre-app.yaml`

Four resources, separated by `---`. `kubeconform -verbose -summary -strict` reported all four valid. Point at `deploy/kubeconform.png` when you say that.

**1. `apps/v1` Deployment `tre-app`**

- `metadata.labels.app: tre-app` and the pod template label `app: tre-app` and `spec.selector.matchLabels.app: tre-app` are the same string. The Service selector uses that same string. If those three differ, the Service has no endpoints.
- `spec.replicas: 2`. Two pods. One can be rescheduled while the other still serves `submit` and `approve`.
- Pod `securityContext`:
  - `runAsNonRoot: true`. The kubelet rejects the pod if the process would run as uid 0.
  - `runAsUser: 10001` and `runAsGroup: 10001`. A concrete non-root uid, so the flag is not only a Boolean.
  - `seccompProfile.type: RuntimeDefault`. The runtime default syscall filter.
- Container `tre-app`:
  - `image: tre-team/app:0.1.0`. Made-up image. No Dockerfile is required for this assignment.
  - `imagePullPolicy: IfNotPresent`.
  - `ports[0].name: http`, `containerPort: 8080`. The Service targets the name `http`, not a hard-coded number, so the Service and the container stay bound if the number moves.
  - Container `securityContext` repeats `runAsNonRoot: true`, sets `allowPrivilegeEscalation: false`, `readOnlyRootFilesystem: true`, and `capabilities.drop: [ALL]`. The process cannot gain root, cannot write the image filesystem, and starts with no Linux capabilities.
  - `resources.requests`: CPU `100m`, memory `256Mi`. The scheduler uses requests for placement.
  - `resources.limits`: CPU `500m`, memory `512Mi`. The cgroup enforces the ceiling.
  - `env` `DB_PASSWORD` from `secretKeyRef.name: tre-db`, `key: password`.
  - `env` `TRANSLATION_API_KEY` from `secretKeyRef.name: tre-api`, `key: token`.
  - There is no `value:` field under those env entries. The YAML in git contains no secret material. The Secret objects themselves are created out of band.

**2. `v1` Service `tre-app`**

- `spec.type: ClusterIP`. The virtual IP is reachable only from inside the cluster. Browsers hit an ingress in front of this Service. The pods are not given a public NodePort or LoadBalancer.
- `spec.selector.app: tre-app`. Matches the Deployment pod label.
- `port: 80`, `targetPort: http`, `name: http`. Clients inside the cluster use port 80. kube-proxy sends that to container port 8080 via the named port.

**3. `autoscaling/v2` HorizontalPodAutoscaler `tre-app`**

- `scaleTargetRef` points at `apps/v1` Deployment `tre-app`.
- `minReplicas: 2`, `maxReplicas: 6`. The floor matches the Deployment replica count. The ceiling is 6.
- Metric: resource CPU, `Utilization`, `averageUtilization: 70`. When the average CPU across pods stays above 70 percent of the request, the controller adds pods, and it stops at 6. This is how the monolith scales under upload load without splitting into services.

**4. `networking.k8s.io/v1` NetworkPolicy `tre-app`**

- `podSelector.matchLabels.app: tre-app`. The policy applies to those pods only.
- `policyTypes: [Ingress, Egress]`. Once both are set, traffic that is not listed is denied.
- Ingress: TCP `8080` only. That is the container port. Port 80 on the Service is cluster networking in front of the pod; the pod itself accepts 8080.
- Egress:
  - UDP `53` and TCP `53`. DNS, or the pod cannot resolve PostgreSQL, the identity provider, object storage, translation, or notification.
  - TCP `5432`. PostgreSQL. This is the `SQL/TLS` arrow on B2.
  - TCP `443`. HTTPS to the identity provider, object storage, translation service, and notification service.

**Close.** "PP-4 is Okoro's requirement that a release has a stored subject, purpose, benefit, decision, and time. That requirement is UC-6 and UC-7, the `AccessRequest` state machine, the activity fork around the audit write, one monolith, and this Deployment. `approve()` commits the audit row in PostgreSQL. The manifest keeps that process non-root, on a private ClusterIP, with the database password outside the YAML."

**If asked**

- A manifest is declarative desired state. The controller reconciles pods to `replicas`, the image, the resource limits, and the security context. We are not shipping application source.
- `readOnlyRootFilesystem: true` means the container cannot write the image layer. Logs and temp files would need an emptyDir volume if we added them later. The assignment image does not require that volume.
- HPA scales pods, not the database. PostgreSQL remains the bottleneck named in B1.
- The NetworkPolicy egress to 443 is shared by four partners. It does not encode which host. Host-level restriction would be a follow-on policy. The trust boundary for those hosts is already drawn on B2.
