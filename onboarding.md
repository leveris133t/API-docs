# IB User Activation

The **user activation service** is responsible for creating a customer account, configuring their profile and activating them on the system.

This is also known as the **onboarding service**.

## Responsibilities of the service

The ultimate responsibility is to create a *`Party`* object on the system. When the onboarding starts, the *`Party`* object for that user is created. As the onboarding progresses, more and more customer data will be added to the *`Party`* object.

The *`Party`* object will ultimately contain, or have relations to:
* **Basic user data** e.g. First name, surname
* **Contact details** e.g. Email, phone number
* **Customer consents** to satisfy compliance requirements
* **Security details** for authentication purposes. This includes passwords, device credentials and biometrics
* **Identity and address documentation** for verification purposes. The details of these documents are automatically extracted by the system and stored

Based on the **Party** data collected, the onboarding process will verify:
* **Contact details** on the email addresses and phone numbers captured and
* **Identity and address documentation** with **KYC** and **AML** checks to satisfy compliance requirements

The process also:
* Facilitates **recapturing party data** in case a verification has failed
* **Activate a user account** when the *`Party`* has sufficient detail. This allows the user's to access further client services

**RADIM RADIM RADIM**

Don't know what this is:
* Defined *`Product applications`* and *`Pricing schemas`* are ready to be used and setup by the Customer.

What does 'ready' mean in this case

**RADIM RADIM RADIM**




## How to use the service

Each onboarding process is comprised of a number of pre-defined steps called the *`Process definition`*.

These steps can include:
* **Form steps** to capture textual information from the user. This can include personal details, contact details, passwords etc. (type: `FORM`)
* **Authentication steps** where contact details and some biometrics are verified (type: `AUTH`)
* **Custom steps** which are used for more specific functionality e.g. KYC and AML processes (type: `CUSTOM`)
* **Application steps** which are used to create products on behalf of the user (type: `APPLICATION`)

The **front-end application** must to follow the *`Process definition`* in order to complete the onboarding.

The *`Process definition`* can also depend on the type of **front-end application** (web or mobile) that the onboarding is started from.

### Starting the onboarding process

1. Call `public/processes/!startProcess` to create a new onboarding process. This will return the id of the new process
2. Call `public/processes/{idProcess}` to get the full details of the current onboarding step
3. Optionally call `public/process-definitions/{idProcessDefinition}` to get the full list of steps (*`Process definition`*)

### Advancing to the next onboarding step

Once the application has fulfilled the requirements of the current step, call: `/processes/{idProcess}/!executeCurrentStep` to move to the next step.

This will return the next onboarding step or notify the front-end that the process is complete.

On occasion, `/processes/{idProcess}/!executeCurrentStep` will not need to be called. This happens as the execution has been performed by the server. This depends on the specific step type.

![How to use the service](onboarding-how-to-use-the-service.png)

### User activation and public and private endpoints

When the onboarding process is started, the user's account will be created, but not activated.

As such, **the user will not have** an *`Authentication token`* to be able to log into the application.

Activation will occur when the *`Authentication token`* is returned in a call to `/processes/{idProcess}/!executeCurrentStep`.

Before this point, all URLs must use the `/public/` endpoints. Afterwards, all URLs must use the `/private/` endpoints and pass in the *`Authentication token`* in the headers.

The account can be activated by the server at any stage after the user's authentication credentials are captured. This point can be configured on the system.

### Supplementary processes

** RADIM RADIM RADIM **

After execution of some specific types of steps we can start one or more complementary processes on background. In order to get list of processes we recommend you to call `/processes/!list` endpoint in `/Onboarding - private part` after every execution of "private" step. Every process has defined a `priority` attribute which defines the order of the processes, where the process with the lowest priority number will be the first.

Some types of steps can have more complex flow to follow before they call the `execute`, see `steps[].stepType` attribute in `/process-definitions` resource and require some step type specific calls before its execution e.g. `Device setup` API.

Some of them are triggered and executed asynchronously on the Back-end so We provides Consumers application with Event about changes in process  - `IB_ONBOARDING_PROCESS_CHANGED` event (see [Asynchronous Communication service](mw-gen-asynccomm-ib.md) to get more information).
