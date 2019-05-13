# IB User Activation

This service manages the activation process of new *Customer*, also known as the *`Onboarding process`*. We use the term Prospect for Customers which have not signed a Contract in Leveris Client.

## Responsibilities of the service

* A *`Party`* is created so that we can connect all information with the particular person.
* Required Party data and *`Consents`* are collected so that we fulfill compliance requirements.
* Customer *`User account`* with the required credentials is activated so that the *`Customer`* is able to use the digital channel to access and manage the Client services.
* *`Contact details`* (phone number, email address) are verified so that contact details exist and are accessible by the Customer.
* Other *`Security details`* (password, device, biometric, etc.) credentials are registered so that the customer can use them for authentication.
* Party *`Identity verification`*, *`KYC check`* and *`AML check`* are processed so that we fulfill compliance requirements.
* Party *`Address`* is collected and verified so that is has the proper relation to the customer.
* Defined *`Product applications`* and *`Pricing schemas`* are ready to be used and setup by the Customer.


## General description

Onboarding is comprised of a number of steps and is driven by the Customer. It has to be configured by Leveris team which steps are included into onboarding process. Basic principle is that consumer application have to follow the pre-defined flow of steps. We call it `/process-definitions`. Some steps within process definition make sense only for specific consumer app channel - `ProcessChannel` .
Execution of some steps of on-boarding process does not need to require User authentication. It is also matter of step configuration whether user needs to be authenticated or not. 
For steps which require an authentication `/Onboarding - private part` API needs to be used and `/Onboarding - public part` API for the rest.
It can be also configured when Customer User account is activated during onboarding process however be aware that it has to be activated after the credentials suitable for the User authentication are collected.

In order to start onboarding process You, at first, need to get and choose `/process-definitions` in `/Onboarding - public part` API, and call `/processes/!startProcess` endpoint. Once the process is started We provide you information about what is next step within the process to execute. In order to execute step use `.../processes/{idProcess}/!executeCurrentStep`. When the step is executed, we validate that required conditions for the given step are met. Once the execution is successful We return information about what is next step within the process to execute. 
After execution of some specific types of steps We can starts one or more complementary processes on background. In order or get list of processes We recommend you to call `/processes/!list` endpoint in `/Onboarding - private part` after every execution of "private" step. Every process has defined `priority` field which determines a processing order of the processes. 

Some types of steps can have more complex flow to execute, see `steps[].stepType` field in `/process-definitions` resource and require some step type specific calls before its execution e.g. `Device setup` API. 
Some of them are triggered and executed asynchronously on the Back-end so We provides Consumers application with Event about changes in proces  - `IB_ONBOARDING_PROCESS_CHANGED` event (see [Asynchronous Communication service](mw-gen-asynccomm-ib.md) to get more information).
