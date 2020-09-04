## Savings

Each `DepositProduct` can have several associated `Saving` goals.
A `Saving` facilitates a user in reaching a target/goal amount.
The user has the ability to fund the account, setup recurring funding and can avail of the rounding up scheme ([Rounding up service](roundingup.md)).
The user can also withdraw funds at any time.


Via the service a user is able to:

-   [Create, edit and close a saving goal](#create-edit-and-close-a-saving-goal)
-   [Fund or withdraw a saving goal](#fund-or-withdraw-a-saving-goal)
-   [Manage a recurring funding](#manage-a-recurring-funding)
-   [Manage round up](#manage-round-up)

### Create, edit and close a saving goal

To create or edit a `Saving`, the user needs to provide a name and target amount and, optionally, the user can introduce a target date.
All of these attributes do not introduce any obligation to be completed, and they are merely to help the user with achieving a goal.

Closing a `Saving` can be done anytime, any money funded will be automatically withdrawn into the holding `DepositProduct`.

### Fund or withdraw a saving goal

To fund a `Saving`, a user can simply pass in the amount and the currency from where the money should be transfer from.
Note that the currency needs to be one of the supported by the holding `DepositProduct`, and sufficient amount has to be available.

To withdraw, a user can only choose the amount to be withdrawn. The money will be transferred to the holding `DepositProduct`'s currency account of the same currency of the `Saving`.

### Manage a recurring funding

Using this feature, a user can set up or edit a recurring funding. To do it a user needs to provide the information about how much should be fund, what's the periodicity (week, month, etc) and dates of next fund or last. Note that the date of next fund has to be set to at least the next day.

A user can cancel the recurring funding at any time.

### Manage round up

A user can accelerate the savings in this `Saving` goal by setting up rounding up feature.
With this feature a user will be able to automatically transfer a rounded up amount of the transactions performed in the holding `DepositProduct`.
e.g. a transaction of €1.90 will fund the `Saving` with €0.10

To know more about rounding up see: [rounding up](roundingup.md)
