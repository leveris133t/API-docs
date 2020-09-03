This service provides all information about each deposit product of a client of the bank. It also enables management of the deposit product and its transactions.

## Responsibilities of the service

* Details about a [deposit product](#deposit-product), its status, currency components, balances and holders. It allows to change the name of the deposit product and add or remove `currency components`.
* List of [transactions](#transactions-and-required-transactions) for a `deposit product` with a possibility to add a note to each transaction, [create payment order](#payment-order-creating) or conversion between currency components.
* Balance and list of [upcoming payments](#upcoming-payments) and [deferred payments](#deferred-payments) with an option to cancel an upcoming payment.
* [Authorization of payment orders](#authorization-of-payment-orders).

## Deposit Product

A deposit product is created during the onboarding process (see [User Activation](mw-gen-user-activation-ib.md) service).
Its parameters are set according to `deposit product type` selected by the user when created.
A client can have more then one deposit product, list of which can be obtained by endpoint `/deposit-products/!list`
in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/). Active operations like changing currency components
or creating transactions can only be made on `deposit product` in **ACTIVE** state.

Currency components of `deposit product` can be activated or deactivated within the bounds of `deposit product type`.
One currency component has an exceptional position and that is the component of primary currency. It cannot be deactivated.
Also aggregated balance is returned in the primary currency.
It is possible to change primary currency by setting it as the currency with highest priority with `/deposit-products/{id}/currencyPriorities` in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).

To gather all information about a deposit product you might want to call these endpoints in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/):

1. Detail of deposit product at `/deposit-products/{idProduct}` endpoint to get the name, currency components, holders and aggregated balance in primary currency.

2. `/deposit-products/{idProduct}/!getBalance` for each active currency component to get individual currency balances.

3. `/deposit-products/{idProduct}/!getUpcomingSummary` and `/deposit-products/{idProduct}/!getDeferredSummary` to obtain sum of [upcoming payments](#upcoming-payments) and  [deferred payments](#deferred-payments).

4. `/deposit-products/{idProduct}/!getUnauthorizedPaymentsSummary` to obtain number and total sum of [unauthorized payments](#payment-order-po-entity).

It is possible to send user account information to an email address with `/deposit-products/{idProduct}/!emailAccountInfo` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).

## Internal credit transfer

Internal credit transfer is a transaction between two internal accounts which are both on the Leveris Platform.
To create internal credit transfer you should call endpoint `/deposit-products/{idProduct}/!createInternalCreditTransfer` from [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/)
with request which should contain attributes:
* **Creditor identification type** - the creditor's account can be identified by three possible ways:
  * Account number - the creditor's account number in IBAN format.
  * E-mail address - the creditor's e-mail address.
  * Phone number - the creditor's phone number in international format.
* **Creditor identification** - the creditor identification value (e.g. IBAN, email address, phone number). This attribute must correspond to the chosen **creditor identification type**.
* **Amount** - amount of money which will be sent to the creditor.
* **Due Date** - date when the transaction will be processed. When this field has no value then the transaction will be processed immediately.
* **Message for creditor** - the debtor can use this attribute for send simple text message to the creditor.

When internal credit transfer is successfully created you will receive response which contains identification number of the transaction and the operation status=`OK`.
Otherwise you will get response with description of the reason why the internal credit transfer was not created.

## Transactions and Required Transactions

There are two types of transactions available in separate endpoints of [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/):

* `/deposit-products/{id}/transactions/!search` for `transactions` (also called `processed transactions`). These are transactions that were performed on the account including necessary auto conversions between currency components to satisfy a payment order.
* `/deposit-products/{id}/required-transactions/!search` for `required transactions`. These are transactions in a form in which a client requested them eg. transfer of money in some currency to another account. Such a transfer might have incurred automatic conversions between currency components.

For each `processed transaction` there is just one `required transaction`, but if there are automatic currency conversions there will be several `processed transactions` for a single `required transaction`.
For convenience sake a detail of a required transaction in `/deposit-products/{idProduct}/required-transactions/{idTransaction}` endpoint of [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/) contains
the most important attributes of related `processed-transactions` in `breakdown` attribute.

Included within transaction lists are reservations made by card processors, they have status **RESERVED**.
Revoked transactions have status **CANCELLED**, all other transactions should have status set to **PROCESSED**.

Details of merchant on which behalf the reservation was made can be found in `extendedAttributes`.
List of all potential `extendedAttributes` is available from `/transactions/!extendedAttributesDefinition` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).

## Upcoming Payments

Scheduled payments can be obtained by `/deposit-products/{id}/upcoming-payments/!search` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).
The result can also contain payments which were just created and have not yet been processed.

## Deferred Payments
Payments which have not been processed yet and should have been are available in `/deposit-products/{idProduct}/deferred-transactions/!search` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit-ib/deposit-ib/latest/).
This usually happens because of insufficient funds in the `deposit product`.

## Authorization of payment orders

### Payment order (PO) entity

Payment order (PO) is prescription of payment and from the business perspective Leveris platform distinguish two types of payments on Leveris platform:
- Internal payments - payments between two accounts which are both on the Leveris platform
- External (outgoing) payments - payment from internal account within Leveris platform to external account outside Leveris platform.

In current state, authorization of PO in IB is only provided for external outgoing payments and internal payments are not authorized.

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
Each attribute has name and for example information about his validation (optional or mandatory, maximal length etc.).

### Payment order creating
For the correct payment order creation, we need provide these steps:

1. Call `/api/private/deposit/deposit-products/{idProduct}/payments/!getForm` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit/deposit-ib/latest). This endpoint returns correct PaymentForm entity by input:
    - account number
    - amount
    - country

2. Call `/api/private/deposit/deposit-products/{idProduct}/payments` endpoint in [Deposit API](/book/mw-ib/mw-gen-deposit/deposit-ib/latest).
This endpoint checks whether the input data for making the payment order corresponds with the PaymentForm entity returned in the first step.
If everything is alright, payment order is created.

### Authorization process
External payment order is created in status PENDING APPROVAL.
- In this status, payment order can be authorized by the customer and becomes **PROCESSING** or **SCHEDULED** (based on due date).
- Customer is also able to cancel this PO and PO becomes **CANCELED**.

All other statuses and transition between statuses of PO are not relevant from the authorization point of view.

Example of payment order's authorization process:

1. Calling endpoint for payment order approving, please see `/api/private/deposit/deposit-products/{idProduct}/unauthorized-payments/{idPayment}/!approve`
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

### Provided functionality

In the area of unauthorized payments, IB provides these services:

- overview of user's payment orders
- ability to approve payment order
- ability to cancel payment order

## Payee management

Payees allow users to send payments quickly and comfortably to their friends, colleagues, relatives, etc. A ‘Payee’ is basically a payment template that can be used to fill in details about an outgoing payment, i.e. users can use a stored Payee to initiate a payment with the given payee details.

The user can save the payee’s payment details after creating a payment or the user can create a payee before creating the payment and then use the payment details saved within the payee to create the payment.

Payees are always for each user (CIF ID) individually and are not shared among different users.

To create a new payee, we rely on the payment method, SmartPayment forms definitions and configurations that help to choose the right payment form based on the payee’s country and currency. Optionally, the transaction amount can be used to determine a more suitable form for the payee.

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
