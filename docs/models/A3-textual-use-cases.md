# A3. Textual use cases

The two use cases below are the walkthrough for PP-4. The use case diagram still carries PP-1, PP-2, PP-3, and PP-5.

## UC-6 Request dataset access

**Caption:** This use case supports PP-4. It creates the record of who wants a dataset and why.

| Row | Detail |
|---|---|
| Actors | Researcher (Dr. Ezer Kang). Identity provider. TRE, through AccessRequest, Dataset, and AuditRecord. |
| Description | The researcher asks to receive the accepted analysis file for one published dataset. He must state the purpose and the expected benefit to the people the data came from. |
| Data | Session, datasetId, purpose, benefit. |
| Stimulus | The researcher submits the access-request form. |
| Response | The TRE returns a requestId. The request status is Submitted. An audit record stores the action submitted. |
| Pre/post-conditions | Pre: the researcher has a session, and the dataset exists. Post: an AccessRequest is Submitted, with purpose and benefit stored. If an alternative flow runs, no Submitted request is kept. |
| Comments | Supports PP-4. This is the first half of the class walkthrough. The source upload is not sent to the researcher at this step. |
| Alternative flows | 1. Purpose is empty, or Dataset.isPublished() is false: AccessRequest returns rejected and writes no submitted audit record. 2. The researcher calls withdraw() before review: AuditRecord.record(withdrawn) runs, and the status becomes Withdrawn. |

## UC-7 Review access request

**Caption:** This use case supports PP-4. It stores the custodian decision and tells the researcher.

| Row | Detail |
|---|---|
| Actors | Data custodian (Daniel Okoro). TRE, through AccessRequest and AuditRecord. Notification service. |
| Description | The custodian reads a submitted request and approves or denies it. Approval releases the accepted analysis file to that researcher. The source file stays in object storage. |
| Data | requestId, decision, deny reason, custodian identity, decidedAt. |
| Stimulus | The custodian opens a submitted request and chooses approve or deny. |
| Response | The status becomes Approved or Denied. AuditRecord stores that decision with the custodian and the time. NotificationService.notify tells the researcher. On approval, the researcher can take the accepted analysis file. |
| Pre/post-conditions | Pre: the custodian has a session, and the request is Submitted. Post: the decision, the actor, and the time are stored, and the researcher has been notified. |
| Comments | Supports PP-4. This is the second half of the class walkthrough. Only a user in the Custodian role can approve, deny, or revoke. |
| Alternative flows | 1. The status is already Withdrawn: AccessRequest returns closed, and no decision is stored. 2. The custodian denies the request but leaves the reason empty: AccessRequest returns rejected, and the status stays InReview. |
