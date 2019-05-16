# Cards

The cards service is responsible for allowing a user to create, activate and manage their card(s).

## Responsibilities of the service

Ultimately the cards service is resposible for:
* **Creating cards** and other associated activities e.g. setting delivery addresses, activating the card and setting the PIN
* **Card security settings** e.g. the types of transactions allowed and limit management
* **Managing the card lifecycle** e.g. Reporting cards lost or stolen, freezing/unfreezing and card cancellation
* **Managing card attributes** such as the PIN value

There are two types of cards that are supported by the cards service:
* **Virtual cards**: Virtual cards are the most basic representation of a card with a card number, expiry date and CVV/CVC. They can be used for e-commerce transactions and, when supported, with NFC (Near-Field-Communication) capable smartphones
* **Physical cards**: Physical cards have an associated plastic card which can be used at ATMs and POS's (Points of Sale). Every physical card must be created by an external card provider and subsequently delivered to the user by post or handed over in a physical branch

## How to use the service

The cards service requires that the user has been onboarded and is logged in to the Internet Banking service.
The back office ultimately defines how many cards, and which type of the cards, the user is able to own.

Each card has an associated status. From this, we can determine what to communicate to the user and what actions the user is able to perform. Please see [Card statuses](#card-statuses) for more details.

Using the cards service the user is able to:

-   [Order a card](#order-a-card)
    -   [Getting started](#getting-started)
    -   [Creating the card](#creating-the-card)
    -   [Setting the PIN (physical cards only)](#setting-the-pin-physical-cards-only)
    -   [Activating the card (physical cards only)](#activating-the-card-physical-cards-only)
-   [View secure card details like its number and CVV/CVC](#view-secure-card-details)
-   [Freeze/unfreeze the card](#freezeunfreeze-card)
-   [Manage security and limits of card usage](#card-limits-and-security)
-   [Cancel the card](#cancel-card)

For physical cards, the user of the cards service can also:

-   [Change the PIN](#change-physical-card-pin)
-   [Report the card as lost, stolen or detained (damaged or broken)](#report-physical-card)

### Card statuses

- **Ordered**: The card has been created on the system and is ordered. For virtual cards this status is momentary and doesn't require any further action from the user. For physical cards, the user will be able to track the card's delivery and activate it when the card has been received
- **Waiting for 1st PIN authorised transaction**: The card was activated in the app but needs a PIN transaction in an internet connected POS or ATM. Physical cards cannot be used for transactions while this status is active. Virtual cards are unaffected by this status
- **Active**: The card is fully active
- **Frozen**: The card is temporarily blocked. This can be triggered by the user or by the bank. When frozen by the user, the user will be able to unfreeze through the cards service. When frozen by the bank, it can only be unfrozen via the back-office interface
- **Permanently blocked**: The card was reported as lost, stolen or detained and is now permanently blocked. It can't be used for any type of transaction i.e. the card is effectively cancelled

![State diagram for the card statuses](card_statuses.png)

Note that after performing an action that can update the status ([activate](#activating-the-physical-card), [freeze](#freeze), etc), the actual status of the card may take a while to update. For these cases, it's recommended to use polling until the status has been updated to the expected status.

### Order a card

#### Getting started

To create a virtual card or to order a physical card the user needs to indicate which account the card will be associated with. To get the list of accounts we can use the [/accounts/!list](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#1132) endpoint.

The user also needs to choose one of the available card products. The card product technology type defines which type of card will be created: **virtual** for virtual cards and **contactless** for physical cards. To get the list of available card products we can use the [/products/!list](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#837) endpoint.

In the case of ordering a physical card, the user has the option to provide a delivery address. If a delivery address is not provided, the cards service will use the address registered during the onboarding process. To get the list of available delivery addresses we can use the [/delivery-addresses/!list](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#1121) endpoint.

![Obtaining cards initial values diagram](create_card_init.png)

#### Creating the card

We can use the [/cards/!create](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#874) endpoint passing the required **accountNumber** and **productName**. If creating a physical card, we have the option to specify the **deliveryAddress**.

After the creation of the card with the cards service, we need to contact our external card provider to obtain the identifier of the card in the external card provider's service, the **s2cCardId**. To do that we need to use the **externalCustomerId** to get a token, through the [/cards/!token](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#848) endpoint, so we can get the list of cards registered in the provider's service. We then use this list to find the card with the same **externalCardId** and get its **s2cCardId**. We then need to register this with the cards service through the [/cards/{cardId}/!setS2cCardId](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#1086) endpoint.

See the sequence diagram below:
![Create card diagram](create_card_create.png)

#### Setting the PIN (physical cards only)

When creating a physical card, we have to associate a PIN. We can do this after obtaining the **s2cCardId**.
After the user provides the PIN, we need to request a new token to contact the external card provider's service. We then can use the token to create the PIN using the provider's service. Note that the cards service will never request or store the PIN directly, this is only allowed when using the external card provider's service directly.

When the PIN is created on the external card provider's service, we have to tell the cards service that this operation was finished successfully. To do that, we can use the [/cards/{cardId}/!updatePinFlag](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#1100) endpoint.

See the sequence diagram below:
![Setting the PIN on a physical card diagram](create_card_set_pin.png)

#### Activating the card (physical cards only)

After ordering a physical card, the user has the option to activate the card when the card is delivered. Up until this point, the card cannot be used for transactions. To activate, we can use the [/cards/{cardId}/!activate](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#915) endpoint.

### View secure card details

The cards service doesn't provide the secure details (Full card number, CVV/CVC) directly, but we can get these values from the external card provider's service.
To do that we need to obtain a token for accessing the provider's service using the [/cards/!token](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#848) endpoint. We can then access the provider's service to obtain these details.

### Change physical card PIN

The cards service never requests or stores the PIN of the card. To change the PIN of the user's physical card we need to do it through the external card provider's service.
To do that we need to obtain a token for accessing the provider's service using the [/cards/!token](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#848) endpoint. We can then access the provider's service to change the PIN.

Note that after a PIN change, the user is required to make a PIN transaction with their physical card at an ATM or POS.

<!-- TODO add new refactor -->

### Report physical card

The user can report a physical card as lost, stolen or detained (damaged or broken). To do this we can use the [/cards/{cardId}/!block](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#932) endpoint with the blockage type **PERMANENT**.

Note that this action can't be reverted and the user will not be able to use the card anymore after this action.

### Freeze/unfreeze card

#### Freeze

The user is allowed to freeze a card. When the card is frozen the card is blocked and no transactions can be made.
To freeze the card we can use the [/cards/{cardId}/!block](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#932) endpoint with the blockage type **TEMPORARY**.

#### Unfreeze card

When the card has been frozen by the user, the user is allowed to unfreeze it. To unfreeze the card we can use the [/cards/{cardId}/!releaseBlockage](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#952) endpoint.

### Card limits and security

#### Security

The user is able to enable or disable security features that the card may support e.g. contacless payments, magstripe payments etc. via the `/cards/{cardId}/!setSecurity` endpoint.

#### Enable e-commerce

To enable or disable e-commerce we can use the [/cards/{idCard}/!updateEcommerce](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#1031) endpoint and pass in a boolean with the desired option.

#### Limits

The default card limits are defined at the card product level, which means all cards are issued with predefined limits.

All limits are defined based on the same time period:

-   **Daily** - limit is defined by maximum amount or number of transactions per day
-   **Weekly** - limit is defined by maximum amount or number of transactions per week
-   **Monthly** - limit is defined by maximum amount or number of transactions per month

Each limit can be defined by a maximum amount or by the number of transactions

The user is allowed to change each limit up to a maximum limit. The maximum limit value can be obtained from each limit definition on the card details from the [/cards/{cardId}](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#902) endpoint.

##### Permanent limit

Changing the permanent limit will change the default limit forever. This means it can never be reset to the original value (unless the original value is passed in as a value to the endpoint).
To change the permanent limit(s) we can use the [/cards/{cardId}/!setPermanentLimits](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#1011) endpoint. This endpoint can change multiple limits at the same time.

##### Temporary limit

The user has the option to create a temporary limit that will stay valid until the provided date. To create a temporary limit(s) we can use the [/cards/{cardId}/!setTemporaryLimits](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#991) endpoint. This endpoint can change multiple limits at the same time.

##### Remove temporary limit

The temporary limits can be removed at any time. This action will reset the limit to its permanent value. To remove a temporary limit(s) we can use the [/cards/{cardId}/!setTemporaryLimits](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#991) endpoint. We must pass an array of the temporary limit **name** without the limit **value** and **validTo** date as a parameter. This endpoint can change multiple limits at the same time.

### Cancel card

If the user wants to cancel the card, we can use the [/cards/{cardId}/!cancel](https://doc.ffc.internal/api/mw-gen-payment-card-ib/payment-card-ib/latest/#docs/method/#966) endpoint.

Note that this action can't be reverted and the user will not be able to use the card anymore after this action.
