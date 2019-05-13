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
* Facilitates **Recapturing party data** in case a verification has failed
* **Activate a user account** when the *`Party`* has sufficient detail. This allows the user's to access further client services

**RADIM RADIM RADIM - TO CLEAR UP**

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

### Handling the process steps

#### Form steps

The information to capture is defined in *`currentStep.formProperties`*. It must be validated by the front-end and then submitted to `/processes/{idProcess}/!executeCurrentStep`.

#### Auth steps

Before this step can be executed, we typically have to follow this flow:
1. Call `processes/{idProcess}/!startAuthSubprocess` to start the auth process. This will send a verification message in the case of SMS or email and return the identifier of the sub process (*`idAuthProcess`*)
2. Call `processes/{idProcess}/auth-processes/{idAuthProcess}/!validateCurrentAuthSubprocessStep` with the SMS or email code to validate the auth process

![How to complete an auth step](onboarding-auth-process.png)

#### Custom steps

The workflow for this step depends on the *`step.code`*.

The codes can be:
* **JUMIO** for the Jumio Proof of Identity service (see [documentation here](mw-gen-jumio-ib.html.md))
* **JUMIO_POA** for the Jumio Proof of Address service (see [documentation here](mw-gen-jumio-ib.html.md))
* **KYC** and **ADDRESS_APPROVAL**. These are waiting steps for the front-end while the back-end contacts 3rd parties. See [back-end notifications](#back-end-notifications)
* **DEVICE_CREDENTIALS** for regitering a device. See the `Device setup` API as part of the onboarding documentation
* **AGREEMENT** for terms and conditions

#### Application steps

Based on the back-end configuration, the front-end application may have to create a certain type of default product for the user. For further information please look at the deposit API.

### User activation and public and private endpoints

When the onboarding process is started, the user's account will be created, but not activated.

As such, **the user will not have** an *`Authentication token`* to be able to log into the application.

Activation will occur when the *`Authentication token`* is returned in a call to `/processes/{idProcess}/!executeCurrentStep`.

Before this point, all URLs must use the `/public/` endpoints. Afterwards, all URLs must use the `/private/` endpoints and pass in the *`Authentication token`* in the headers.

The account can be activated by the server at any stage after the user's authentication credentials are captured. This point can be configured on the system.

### Back-end notifications

At some stage in the process, the back-end may need to notify the front-end about either:
* A **change in the onboarding process** e.g. the current process on the back-end has completed or
* A **new complementary process** has been created by the back-end

When either of the above happens, an `IB_ONBOARDING_PROCESS_CHANGED` event is served asynchronously via web socket. See [Asynchronous Communication service](mw-gen-asynccomm-ib.md) for more details.

### New complementary processes

As the back-end processes the submitted onboarding data, various checks will occur.

When a check fails, the back-end can require further information from the user. This is done by creating a separate process such that the original onboarding process remains unchanged. This can only occur after the *`Authentication token`* has been issued.

To get the list of processes, call `private/processes/!list`. Each process has a defined `priority` attribute which determines which process to complete first. The lowest `priority` value should be completed first i.e. 0 is more important than 1.

We recommend that the front-end calls `private/processes/!list`:
* When a **back-end notification** comes through
* After **login** (see [Router](mw-gen-router-ib.md))
