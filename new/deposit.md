The deposit service lets users view and manage their deposit product and related transactions.

## Responsibilities of the service

- View [deposit product](#deposit-product) details i.e account status, currency components, balances and mandate holders.
- Manage [deposit product](#deposit-product) i.e change account name, add/remove `currency components` and add/remove 'mandate holders' etc.
- View [transactions](#transactions-and-required-transactions) i.e transaction status, debitor and remittance information etc.
- View [upcoming payments](#upcoming-payments) and [deferred payments](#deferred-payments) with option to cancel.
- Create and [authorise payment orders](#authorization-of-payment-orders).
- Create and manage [payee's](#payee-management).

## Deposit Product

A deposit product is created during the onboarding process (see [User Activation](mw-gen-user-activation-ib.md) service).
It's parameters are set according to `deposit product type` selected by the user when created.
A client can have more then one deposit product, list of which can be obtained by endpoint `/deposit-products/!list`
in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/). Active operations like changing currency components
or creating transactions can only be made on `deposit product` in **ACTIVE** state.

Currency components of `deposit product` can be activated or deactivated within the bounds of `deposit product type`.
One currency component has an exceptional position and that is the component of primary currency. It cannot be deactivated.
The aggregated balance of the account is returned in the primary currency.
It is possible to change the primary currency, see `/deposit-products/{id}/currencyPriorities` in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).

### Retrieving deposit product information

1. Complete deposit product detail can be found by calling `/deposit-products/{idProduct}` endpoint. Contains name, status, currency components, mandate holders and aggregated balance information.

2. Call `/deposit-products/{idProduct}/!getBalance` on each active currency component to get individual currency balances.

3. Call `/deposit-products/{idProduct}/!getUpcomingSummary` and `/deposit-products/{idProduct}/!getDeferredSummary` to obtain sum of [upcoming payments](#upcoming-payments) and  [deferred payments](#deferred-payments).

4. Call `/deposit-products/{idProduct}/!getUnauthorizedPaymentsSummary` to obtain number and total sum of [unauthorized payments](#payment-order-po-entity).

Account information can be shared via email by calling the `/deposit-products/{idProduct}/!emailAccountInfo` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).

## Internal credit transfer

Internal credit transfer is a transaction between two internal accounts on the Leveris Platform.
To create internal credit transfer call the `/deposit-products/{idProduct}/!createInternalCreditTransfer` endpoint from [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/)
with relevant request data:
* **Creditor identification type** - the creditor's account can be identified by three possible ways:
  * Account number - the creditor's account number in IBAN format.
  * E-mail address - the creditor's e-mail address.
  * Phone number - the creditor's phone number in international format.
* **Creditor identification** - the creditor identification value (e.g. IBAN, email address, phone number). This attribute must correspond to the chosen **creditor identification type**.
* **Amount** - amount of money which will be sent to the creditor.
* **Due Date** - date when the transaction will be processed. When this field has no value then the transaction will be processed immediately.
* **Message for creditor** - the debtor can use this attribute for send simple text message to the creditor.

Upon successful creation the response will contain the identification number and status of the transaction.

## Transactions and Required Transactions

There are two types of transactions available:

* `Processed transactions` are transactions that were performed on the account including necessary auto conversions between currency components to satisfy a payment order. Processed transactions are obtained by calling the `/deposit-products/{id}/transactions/!search` endpoint.
* `Required transactions` are transactions in a form in which a client requested them eg. transfer of money in some currency to another account. Such a transfer might have incurred automatic conversions between currency components. Required transactions are obtained by calling the `/deposit-products/{id}/required-transactions/!search` endpoint.

For each `processed transaction` there is just one `required transaction`, but if a transaction contains automatic currency conversions there will be several `processed transactions` for a single `required transaction`.

Each transaction has a status. The **RESERVED** status are reservations made by card processors. Revoked transactions will have a **CANCELLED** status and all other transactions will have **PROCESSED** status.

Details of merchant on which behalf the reservation was made can be found in `extendedAttributes`.
List of all potential `extendedAttributes` is available from `/transactions/!extendedAttributesDefinition` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).

## Upcoming Payments

Scheduled payments refers to payment can be obtained via `/deposit-products/{id}/upcoming-payments/!search` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).
Upcoming payments can also contain payments that have just been created and yet to be processed.

## Deferred Payments
Deferred payments refers to payments that could not be processed by the system. This usually occurs when a user has insufficient funds in their account. Deferred payments list can be obtained via `/deposit-products/{idProduct}/deferred-transactions/!search` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).


## Authorization of payment orders

### Payment order (PO) entity

A Payment order (PO) is a prescription of a payment. The Leveris platform distinguish between two types of payments:
- Internal payments - payments between two accounts on the Leveris platform
- External (outgoing) payments - payment from internal Leveris account to external account (outside Leveris platform).

Authorization of PO's is only provided for external outgoing payments. Internal payments are not authorized.

Payment order entity have following statuses:
- **PENDING APPROVAL** - newly created PO and not yet authorized,
- **SCHEDULED** - authorized PO which is postponed in realization by setting due date in the future,
- **PROCESSING** - authorized PO which is being processed the transaction core,
- **PROCESSED** - authorized and processed PO with the settled transaction,
- **CANCELED** - canceled PO before it is PROCESSED (so, no transaction exists),
- **REJECTED** - PO can be rejected from processing for different reasons (eg. insufficient funds),
- **INVALIDATED** - PO can be canceled after processing, so settled transaction is invalidated a status of PO becomes also INVALIDATED.

### PaymentForm (PF) entity
A payment form is an ordered list of payment attributes based on a payment method.
Each attribute has name and a validation information (optional or mandatory, maximal length etc.).

### Payment order creating
To create a payment order:

1. Call `/api/private/deposit/deposit-products/{idProduct}/payments/!getForm` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit/deposit-ib/latest). This endpoint returns correct PaymentForm entity by input for given `account number`, `amount` and `country`.

2. Call `/api/private/deposit/deposit-products/{idProduct}/payments` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit/deposit-ib/latest).
This endpoint ensures the inputted data corresponds with the PaymentForm entity returned in step one. If everything is correct the payment order is created.

### Authorization process
External payment orders are created with status **PENDING APPROVAL**.
- In order to complete the PO, the customer must authorize it.
- Customer also has the ability to cancel the PO.

Example of payment order's authorization process:

1. Approve the payment by calling `/api/private/deposit/deposit-products/{idProduct}/unauthorized-payments/{idPayment}/!approve`
in the [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/)

2. Authorization is provided by **Authorization and Authentication** application, for more information, please see `/auth/auth-processes/{idAuthProcess}/!start` in
[MW Auth API - IB](/book/mw-ib/mw-gen-auth-ib/auth-ib/latest) and `/auth/auth-processes/{idAuthProcess}/!validateStep` also in [MW Auth API - IB](/book/mw-ib/mw-gen-auth-ib/auth-ib/latest).

3. Catch response of `/api/private/deposit/deposit-products/{idProduct}/unauthorized-payments/{idPayment}/!approve` endpoint.
    - Response:
      - `HTTP code 201` with status that describe result of payment order approving:
          - **OK** - payment order was successfully processed
          - **AUTH_FAIL** - authorization of payment order failed
          - **AUTH_NOT_POSSIBLE** - authorization of payment order isn't possible because authorization scenario doesn't exists
          - **INVALID_PAYMENT** - payment order with the given ID does not exist
          - **UNUSABLE_STATE** - payment order is in an unusable state for the requested operation
          - **LOCKED** - payment order with the given ID is locked by another process
          - **REJECTED** - the transaction was rejected by paymentHub (the transaction could not be processed for some reason)
          - **UPDATE** - payment order with given ID does not exist or has already been processed in paymentHub or terminated

## Payee management

Payees allow users to send payments quickly and comfortably to their friends, colleagues, relatives, etc. A ‘Payee’ is a payment template where a user stores the payments details of the recipient for future use. A payee can be created on it's own or when creating a payment.

The payee’s country and currency is required when creating a payee to ensure the correct data is obtained and that the data is in the correct format i.e account number formats vary per country. Additionally, for greater precision in what data is required the amount should also be used i.e for a payment above €10k additional information might be required.

Steps needed to create a new payee:
1. Show the first step with country and currency (amount optional).
2. Determine the right payment form using payment mapping in payment configuration.
    * POST `/payment-forms/!list-by-configuration` in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/)
3. Show the second step with fields from the chosen payment form.
4. Create the payee.
    * POST `/current-user-payees` in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/)

Other supported operations:
* Get a list of payees:
    * POST `/current-user-payees/!list` in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/)
Delete a payee:
    * DELETE `/current-user-payees/{id}` in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/)
