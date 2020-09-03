The **user activation service** allows customers to onboard. It is also known as the **onboarding service**.

## Responsibilities of the service

The user activation service is responsible for:
* Creating a customer's **user account**
* **Configuring their profile**
* **Activating them** on the system
* **Creating the Party object** on the system (see below)

When the onboarding starts, a *`Party`* object for that user is created. As the onboarding progresses, more and more customer data will be added to the *`Party`* object.

The *`Party`* object will ultimately contain, or have relations to:
* **Customer profile data** e.g. First name, surname
* **Contact details** e.g. Email, phone number
* **Customer consents** to satisfy compliance requirements
* **Identity and address documentation** for verification purposes. The details of these documents are automatically extracted by the system and stored
* **User account** This includes security details for authentication purposes i.e. passwords, device credentials, biometrics etc.

Based on the **Party** data collected, the onboarding process will verify:
* **Contact details** on the email addresses and phone numbers captured and
* **Identity and address documentation** with **KYC** and **AML** checks to satisfy compliance requirements

The process also:
* Facilitates **Recapturing party data** in case a verification has failed
* **Activate a user account** when the *`Party`* has sufficient detail. This allows the user to access further banking services

## How to use the service

Each onboarding process is comprised of a number of pre-defined steps called the *`Process definition`*.

These steps can include:
* **Forms** to capture textual information from the user. This can include personal details, contact details, passwords etc.
* **Contact detail verification** where email and phone numbers are verified
* **Product applications** which can either create products on behalf of the user or let users to select products for themselves
* **Other custom steps** which are used for more specific functionality e.g. KYC and AML processes

The exact step types are described in the `currentStepDefinition.stepType` attribute of the `/processes/{idProcess}` endpoint in the [Onboarding - public API](mw-gen-user-activation-ib/user-activation-public-ib/latest/) documentation.

The **consumer application** must to follow the *`Process definition`* in order to complete the onboarding.

The *`Process definition`* can also depend on the type of **consumer application** (web or mobile) that the onboarding is started from.

### Starting the onboarding process

The calls below should be made via the [Onboarding - public API](mw-gen-user-activation-ib/user-activation-public-ib/latest/).

1. Call `/public/processes/!startProcess` to create a new onboarding process. This will return the id of the new process
2. Call `/public/processes/{idProcess}` to get the full details of the current onboarding step
3. Optionally call `/public/process-definitions/{idProcessDefinition}` to get the full list of steps (*`Process definition`*)

### Advancing to the next onboarding step

Once the application has fulfilled the requirements of the current step, call: `/processes/{idProcess}/!executeCurrentStep` to move to the next step.

This will return the next onboarding step or notify the consumer application that the process is complete.

On occasion, `/processes/{idProcess}/!executeCurrentStep` will not need to be called. This happens as the execution has been performed by the server. This depends on the specific step type. For more information see the `stepType` attribute in the [Onboarding - public API](mw-gen-user-activation-ib/user-activation-public-ib/latest/).

![How to use the service](mw-gen-user-activation-ib/onboarding-how-to-use-the-service.svg)

### Handling the process steps

#### Forms

The information to capture is defined in *`currentStep.formProperties`*. It must be validated by the consumer application and then submitted to `/processes/{idProcess}/!executeCurrentStep`.

#### Contact detail verification

Before this step can be executed, the user will receive an SMS or email with a code generated automatically by the system. This code should be used to verify the contact detail by calling `/processes/!verifyContact`. If no SMS or email has been generated, call `/processes/!resendVerification` to generated it manually. 

#### Product applications

Based on the back-end configuration, the consumer application may have to create a certain type of default product (e.g. a current account) for the user. For further information please look at the [Product application service](mw-ldb-product-application-ib.md).

#### Other custom steps

The workflow for this step depends on the *`step.code`*. Please see the [Onboarding API documentation](mw-gen-user-activation-ib/user-activation-public-ib/latest/) for more details.

### User activation and public and private endpoints

When the onboarding process is started, the user's account will be created, but not activated.

As such, **the user will not have** a *`JWT access token`* to be able to log into the application.

Activation will occur when the *`JWT access token`* is returned in a call to `/processes/{idProcess}/!executeCurrentStep`.

Prior to this, all URLs must use the `/public/` endpoints from the [Onboarding - public API](mw-gen-user-activation-ib/user-activation-public-ib/latest/). Afterwards, all URLs must use the `/private/` endpoints from the [Onboarding - private API](mw-gen-user-activation-ib/user-activation-private-ib/latest/) and pass in the *`JWT access token`* in the headers.

The account can be activated by the server at any stage after the user's authentication credentials are captured. This stage can be configured on the system.

### Back-end notifications

At some stage in the process, the back-end may need to notify the consumer application about either:
* A **change in the onboarding process** e.g. the current process on the back-end has completed or
* A **new complementary process** has been created by the back-end

When either of the above happens, an `IB_ONBOARDING_PROCESS_CHANGED` event is served asynchronously via web socket. See [Asynchronous Communication service](mw-gen-asynccomm-ib.md) for more details.

### New complementary processes

As the back-end processes the submitted onboarding data, various checks will occur.

When a check fails, the back-end will require further information from the user. This is done by creating a separate process such that the original onboarding process remains unchanged. This can only occur after the *`JWT access token`* has been issued.

To get the list of processes, call `/processes/!list` in the [Onboarding - private API](mw-gen-user-activation-ib/user-activation-private-ib/latest/). Each process has a defined `priority` attribute which determines which process to complete first. The lowest `priority` value should be completed first i.e. 0 is more important than 1.

We recommend that the consumer application calls `/processes/!list` in the [Onboarding - private API](mw-gen-user-activation-ib/user-activation-private-ib/latest/):
* When a **back-end notification** comes through
* After **login** (see [Router](mw-gen-router-ib.md))
