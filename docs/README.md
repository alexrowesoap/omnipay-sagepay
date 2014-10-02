Network Flow Diagrams
=====================

I have drawn up these diagrams (just one for sagepage-server for now) to help in
understanding the program and data flow while implementing SagePay for a project.

Doing so has thrown up a couple of issues that have been bugging me. They are:

* The final stage, on return from SagePay to your application, is not supported
  by OmniPay at all. That is, there is no abstracted result to check the success
  of the transaction and to extract additional details such as the remote
  transaction reference. The abstraction is kind of "used up" in the notification
  controller. I think the notification controller should use a third, distinct
  set of request and reponse objects.
* The full transaction spans multiple sessions, so the result cannot be passed out
  to the final stage through the session alone. This involves back-end storage.
  I believe it should be possible to abstract the back-end storage in such a way
  that it can be used across all the OmniPay drivers that need this, so it is not
  reinvented over and over. It would then provide a standard way to link the
  results of the transaction sent by SagePay to the notification controller, with
  the final page the user is returned to from SagePay. (This is SagePay Server, where
  the user is sent off to the SagePay site - the iframe version is likely to differ.)

Anyway, take a look at the network flow diagram(s) and see how they work for you.
Feedback would be apreciated.

[Sagepay-Server](https://github.com/judgej/omnipay-sagepay/blob/patch-1/docs/omnipay-sagepay-server.pdf?raw=true) (PDF)
--------------------

Some notes:

* The best place to ensure the result of the transaction request is processed, is the
  first point at which you know the result of the transaction. For SagePay Server, this
  is in the notification contgroller.
* The notification controller also has the important job of responding to SagePay
  to confirm the message has been received and understood. For this reason, the less
  additional processing the notification controller does, the less risk is involved; the
  less there will be to go wrong.
* If there are remote systems to inform of the result of the transaction, for example
  an ERP that needs to be informed what invoices have been paid for, then the best way
  to handle that would be to throw the transaction details into a queue for processing
  independently. For that to work, enough details about the transaction source, e.g. the
  basket or cart being paid for, must be stored alongside the transactino in the back-end
  data store, to enable the queued item to be given enough information to process that
  basket. Bear in mind also that the basket in the user's session is about to be cleared
  as soon as the user is returned to your application by SagePay. It's all about spped and
  reliability of the processing of the transaction result, and timing.

The non-success paths are not shown in the network flow. They will include:

* A notification for an unknown transaction, or for a transaction we have already processed
  a notification for, or a transaction that fails the SecurityKey check. 
  In these cases, we will return an error to SagePay, but will not update
  the transaction details in the back-end storage, since the notification cannot be trustsed
  to have come from a reliable or legitimate source.
