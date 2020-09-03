# Session

This service performs user authentication and other supporting functions.

## Responsabilities of the service

The session service is responsible for:
* **Logging the user in** to authenticate them and return their session
* **Prolonging session tokens** if required
* **Logging the user out**

## How to use the service

* [Logging the user in](#logging-the-user-in)
* [Using the access token](#using-the-access-token)
* [Prolonging the JWT](#prolonging-the-jwt)
* [Logging out](#logging-out)

### Logging the user in

This process will **authenticate the user** and return their access token (`JWT`) with its expiration date.

To log in, follow this flow:

1. Call the endpoint `session/sessions/!getLoginSession` through the [Session - public API](session/public-api) getting the status code `418` with the `idAuthProcess` in the response. This status code notifies us about starting a new AA flow.
2. Complete AA flow according to [AA](aa.md).
3. Call the endpoint `session/sessions/!getLoginSession` providing the `idAuthProcess` and using the [Session - public API](session/public-api). The endpont will return the access token (`JWT`).


### Using the access tokens
