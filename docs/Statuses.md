SagePay Server goes through a number of states, each with a range of statuses.
Most of the statuses are mutually exclusive, but one has two possible meanings.
To work around that, I would propose that the overall status can be stored in a
single transaction record field, but one of the "OK" statuses is mapped to a custom
status so that the status tells us exactly what state the transaction is in.

State: Initialisation

This is the state after SagePay has been sent the initial request to start the transaction.
The return status will be one of the following:

* MALFORMED
* INVALID
* OK
* OK REPEATED

I propose that "OK" is stored as "PENDING" to indicate that SagePay has accepted the
request to start the transaction, and is now waiting for the user to be sent over.
I'm not sure how "OK REPEATING" should be handled yet.

In addition, a custom state of ERROR may be useful to record the event of SagePay
not being contactable.

State: Notification

This state involves SagePay calling the application back with the result of its
interaction with the user. The statuses that SagePay can send are:

* REJECTED
* NOTAUTHED
* ABORT (by user or by SagePay through timeout on abandonment)
* ERROR
* REGISTRATION
* OK

On receipt of a notification, the supplied status would only be stored against the
transaction if the current status is PENDING. The transaction is considered successful
if the status is OK (which highlights why we don't store the OK provided in the
transaction initialisation). ABORT indicates the user has cancelled or abandoned the
payment. All other statuses indicate an error and so a failed transaction.
