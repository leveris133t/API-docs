# IB JUMIO

The **jumio service** is responsible for verifing the customer identity, proving their address and extracting the information from the uploaded files.

## Responsibilities of the service

The service has the responsabilities:
* **Prove the customer identity** and **extract the data** from the file uploaded e.g. ID card, driving license, passport
* **Prove the customer address** and **extract the data** from the file uploaded e.g. Utility bill, bank statement, phone bill

## How to use the service

A *`code`*  will be obtained from the onboarding process (see [Onboarding](onboarding.md)). Based on this you will have:
  * **Proof of identity (POI)** where the customer will upload the files and Jumio will extract the identity data from them (code:  `JUMIO`)
  * **Proof of address (POA)** where the customer will upload the files and Jumio will extract the address data from them (code: `JUMIO_POA`)

The flow to follow in both cases is the same. You only have to change the urls you are using in each case:
  * **Proof of identity (POI)**: `api/private/jumio/poi/` eg: `api/private/jumio/poi/!getScanTransactionStatus`
  * **Proof of address (POA)**:  `api/private/jumio/poa/` eg: `api/private/jumio/poa/!getScanTransactionStatus`

However, the way to handle these cases will be different depending on the channel **web** or **mobile**.

 ### Handling mobile channel

The steps you have to follow to complete the **scan transaction** are:

1. Call `/!getScanTransactionStatus` to know whether the actual status of the transaction was started or not. If the transaction has not started (status: `NOT_STATED_YET` or `PENDING`), move on to the next step. If it's finished (status: `SUCCESS`), go back to the onboarding flow.

2. Call `/!startScanTransaction` with the `processId` to start the scan transaction. This will return you all data you need to open the framework.

3. **Open the framework**. Depending on the data you want to extract, you will use:
 * **Netverify** to proof of identity case (see [iOS](https://github.com/Jumio/mobile-sdk-ios/blob/master/docs/integration_netverify-fastfill.md) and [Android](https://github.com/Jumio/mobile-sdk-android/blob/master/docs/integration_netverify-fastfill.md))
 * **Document verification** to proof of address case (see [iOS](https://github.com/Jumio/mobile-sdk-ios/blob/master/docs/integration_document-verification.md) and [Android](https://github.com/Jumio/mobile-sdk-android/blob/master/docs/integration_document-verification.md))

When the framework has finished, it will be closed and return to the app a `scanReference`. The back-end will use this reference to extract the data.

4. Call `/!setScanTransaction` to send the `scanReference`.
5. Call `/!setScanTransactionStatus` to send the `status`.
6. Go back to the onboarding flow.

![Handling jumio on mobile](jumio-handling-on-mobile.png)

### Handling web channel

The steps you should follow to complete the **scan transaction** are:
1. Call `/getScanTransactionStatus` to know whether the actual status of the transaction was started or not. If the transaction has not started (status: `NOT_STARTED_YET`), move on to the next step. If it's finished (status: `SUCCESS`), go back to the onboarding flow.

2. Get the framework url to start it. Depending on the type of the transaction:
  * Proof of address:
  * Proof of identity:

![Handling jumio on mobile](jumio-handling-on-web.png)

### Frameworks

- Real Jumio
- Mock Jumio
- Web Jumio
