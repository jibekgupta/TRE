# 0. Pain-point scope and traceability

**Product:** Global Health Trusted Research Environment (TRE)
**Course:** Advanced Software Engineering - Model and Architect Your MVP
**Team:** Anu, Ade, Manushi, Jibek

The MVP has two sides. Dr. Amina Kamau uploads a file she already has and records what local terms mean. The TRE drafts an analysis file and can suggest translations for Dr. Ezer Kang. Daniel Okoro decides whether that researcher may receive the analysis file, and the TRE stores the decision. Later diagrams trace to a row in the table below.

## Personas in this MVP

| Persona | Role in the MVP |
|---|---|
| Dr. Ezer Kang | Medical researcher at Howard University. He accepts an analysis-ready file and accepted translations. He receives that file only after a custodian approves his request. |
| Dr. Amina Kamau | Physician and clinical research coordinator at a regional hospital in Kenya. She uploads study data without a separate cleaning project and records local terms. |
| Daniel Okoro | National health-data custodian. He approves or denies access and can see who asked, why, and what was decided. |

Marcus Lee and Bayowa O. are not drivers of this scope. Bayowa's hospital records and board reports are out of scope. Comparing many external datasets inside the TRE is also out of scope.

## Traceability table

| ID | Persona | Pain point | MVP capability | Use case(s) | Success signal |
|---|---|---|---|---|---|
| PP-1 | Dr. Ezer Kang | After a file is exported, he still reformats rows by hand and writes the data dictionary himself before SPSS or R can use it. | The TRE drafts a data dictionary (name, label, type) and a rectangular analysis file from the uploaded table. He edits names and accepts the draft. | UC-4 Prepare analysis file | Every uploaded column has a dictionary row. The accepted file uses those names. He does not build the dictionary by hand on the happy path. |
| PP-2 | Dr. Amina Kamau | Study data sits in spreadsheets, notes, and exports. She has no time to clean it, so it is never shared for research. | She uploads a spreadsheet or CSV she already has. The TRE checks missing values and basic field shape and stores the result as a dataset. | UC-1 Upload dataset; UC-2 Check dataset | She submits an uncleaned file and gets a check report in one sitting. Her active work on that file stays under 15 minutes. |
| PP-3 | Dr. Amina Kamau | Local terms and abbreviations differ by site. A researcher in another country can misread them. | She marks a field or value as a local term and writes what it means. The definition is stored with the dataset and shown with the released analysis file. | UC-3 Document local term | A term she flags cannot be published without a definition. The released analysis file shows that definition beside the field. |
| PP-4 | Daniel Okoro | Sensitive health data can be reused without a clear record of who is using it, why, and how the contributing population benefits. | Every access request names the dataset, the purpose, and the expected benefit. He approves or denies it. The TRE stores his identity and the time. Approval releases the accepted analysis file only. The source upload stays in storage. | UC-6 Request dataset access; UC-7 Review access request | For any dataset he can list requester, purpose, benefit, decision, and timestamp. A request with no purpose cannot be stored. |
| PP-5 | Dr. Ezer Kang | Translating questionnaires and field labels into another language takes significant time. | The TRE suggests a translation of an instrument item or dictionary label. The suggestion is kept only after he edits or accepts it. | UC-5 Translate study text | Each accepted item stores the source text, the accepted translation, the language, and who accepted it. Unreviewed machine text is not published. |

Reserved use cases, reused with these same names in every later diagram:

| ID | Name | Pain point | Relationship |
|---|---|---|---|
| UC-1 | Upload dataset | PP-2 | Base use case for intake |
| UC-2 | Check dataset | PP-2 | Included by UC-1. The check always runs. |
| UC-3 | Document local term | PP-3 | Extends UC-1 when the file has a local term. |
| UC-4 | Prepare analysis file | PP-1 | Researcher accepts the drafted file. |
| UC-5 | Translate study text | PP-5 | Researcher accepts or edits a suggestion. |
| UC-6 | Request dataset access | PP-4 | Researcher submits purpose and benefit. |
| UC-7 | Review access request | PP-4 | Custodian approves or denies. |

## Out of scope

The MVP does not include:

- Finding datasets by search, comparing populations, or analyzing data in a notebook inside the TRE.
- Cultural adaptation of instruments: redesigning answer scales, back-translation as a formal method, or testing whether a translation still measures the same concept.
- Qualitative coding of interview transcripts.
- A hospital operations system: wards, emergency records, paper-chart migration, and reports for a hospital board or a ministry.
- Building KoboToolbox, SPSS, R, or a hospital record system. The researcher uses the released analysis file in the tools he already has.
- Sharing a dataset by email or by any path that skips the access request.
- Automatic approval. A custodian makes the decision.

## Boundary this scope assumes

We build dataset intake, the check report, the data dictionary, local-term definitions, translation review, access requests, and the approval record. We integrate sign-in, file storage, machine translation, and notification. We do not build those four. The context diagram (A1) is the picture of that split.

## Walkthrough thread

The in-class walkthrough follows **PP-4**. Dr. Kang requests access to a published dataset (UC-6). Daniel Okoro reviews that request (UC-7). PP-1, PP-2, and PP-3 are what made the dataset publishable. PP-5 is in the MVP and is not the walkthrough path.

## Architecture constraint

The architecture decision record uses a **modular monolith**: one deployable application with modules for intake, dictionary, translation review, and access. The Kubernetes manifest deploys that one application.
