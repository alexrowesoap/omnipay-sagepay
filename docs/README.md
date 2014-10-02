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

