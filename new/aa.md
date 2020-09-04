# AA

Authorization and Authentication (AA) authenticates a user within a specific flow given them the access they need.

## Responsabilities of the service

The AA service is responsable for:

* **Authenticate a user** using the security data attached by the user.
* **Authorizate a user** to make a specific action.


## How to use the service

The AA flow can be required from the back-end in any request that returns the HTTP status code `418` to the consumer application. At that point, the consumer application has to choose the proper API depending on:
* User wants to *log in* or *reset their password* use the [AA - public API](aa/public-api).
* User *is logged in*. Use the [AA - private API](aa/private-api). In this case, the user is navigating across the app trying to perform any action that requires some authorization on their part as `making a payment`.


When a consumer application receives a notification to initiate an AA flow, it will commonly be said that a **Step-Up authentication** has been fired.

* [Handling AA process](#handling-aa-process)
* [Requested data within the AA process](#requested-data-within-the-AA-process)

### Handling AA proccess

**AA process**, or also called **Step-Up authentication process**, is kicked off by any endpoint within the back-end that returns the status code `418` and the field `idAuthProcess`.

When the consumer application gets the previous status code calling a endpoint, an **AA process** has to be completed to proceed with the action following the next flow:

1. Get a list of the possible scenarios for a given *idAuthProcess* making a `POST` to the endpoint `/auth/auth-processes/{idAuthProcess}/scenarios/!list`.
2. Pick a *scenarioCode* of the previous list and start the **Step-Up authentication process** for the selected scenario using a `POST` to the endpoint `/auth/auth-processes/{idAuthProcess}/!start`.
3. Fill out all the requested information within the consumer application and send it making a `POST` to the endpoint `/auth/auth-processes/{idAuthProcess}/!verifyStep`.
4. If the sent information is correct, the back-end will return the status `FINISHED`. If not, follow API documentation to further details.
5. At this point, the **AA process** flow is completed. Then, the consumer application has to call again the endpoint that requires a *Step-Up authentication process* attaching the *idAuthProcess*. It will return the HTTP status code `200` and the desired information.


### Requested data within the AA process

Each AA process will be comprised of a number of steps that ask for user security details such as password, device ID, OTP, email, and so on. 

If a user does not have sufficient security details to complete the process, a 418 status code will be fired without the `idAuthProcess` field in the endpoint response. If this issue occurs, it should be handled by *Client customer support*.
