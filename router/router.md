# Router

The **router service** allows users to handle their authentication and session.

## Responsibilities of the service

The router service is responsible for:

* **Loging in** the user creating a session for them
* **Prologing the session** of the user
* **Resetting the forgotten password**
* **Loging out** the user
* **Resending the authentication password (OTP)** for the email and SMS

## How to use the service

- [Loging in the user](#loging-in-the-user)

### Loging in the user

This process will authenticate the user and create a session for them (*`Token`*). The calls below should be made via the [Login - public API](https://doc.ffc.internal/book/mw-ib/mw-gen-router-ib/router-login-public-ib/latest/index.html).

1. Call the *`/getLoginScenario`* to get the scenarios to follow. Each scenario is comprised of a number of pre-defined steps.
2. Pick a scenario depending on the requisities eg: user and password, user and biometrics
3. Call the *`/validateLoginStep`* to validate each step of the scenario.





//Reviews:
- Login: This service is responsible for a User authentication using the Party’s User account established during Onboarding process.
- UnLock endpoints. ??
