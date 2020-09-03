# Rounding Up

The Rounding Up services lets users manage the feature of round up payments into their saving goal.

## Responsibilities of the service

Ultimately the Rounding Up service is responsible for:
* **Creating round up details** associated with a specific saving goal in a specific deposit product
* **Change the settings of a round up** e.g. change the multiplier value to speed up or slow down the saving goal
* **Cancelling a round up** for when the user doesn't want to round up anymore

## How to use the service

The Rounding Up service requires that the user has been:
* **Onboarded** and is
* **Logged in** to the Internet Banking service

At any given time, only one active saving can be set up with the round up feature.
Setting the round up to a different saving, will cancel the current one and create a new configuration immediately.


Via the Rounding Up service, the user is able to:

-   [Create a round up setting](#create-a-round-up-setting)
-   [Edit a round up setting](#edit-a-round-up-setting)
-   [Cancel a round up setting](#cancel-a-round-up-setting)


### Create a round up setting

To create a round up setting for a saving goal the user needs to create a `UserSetting` by making a `POST` to the endpoint `/rounding-up/user-settings`.

The user of the endpoint needs to provide the following information:
* Product identifier - `DepositProduct`'s `id`
* Internal account number - `DepositProduct`'s `internalAccountNumber`
* Product type code - `DepositProduct`'s `productTypeCode`
* Saving goal identifier - `Saving`'s `id`
* Saving goal name - `Saving`'s `name`
* Saving goal account number - `Saving`'s `accountNumber`
* Saving goal currency - `Saving`'s `targetAmount`'s `currencyCode`
* Payment rule - the multiplier value from `X1`, `X2`, `X3` or `X5`


### Edit a round up setting

To edit a round up setting, simply make a `PATCH` to the endpoint `/rounding-up/user-setting/{id}` with the information about the new multiplier and the saving goal information.


### Cancel a round up setting

To cancel a round up setting use the `DELETE` HTTP method using the endpoint `/rounding-up/user-setting/{id}`.
