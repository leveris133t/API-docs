# Router

The **router service** allows users to handle their authentication and session.

## Responsibilities of the service

The router service is responsible for:

* **Loging in** the user to authenticate them and return their session
* **Prologing the session** of the user
* **Resetting the forgotten password**
* **Loging out** the user
* **Resending the authentication password (OTP)** for the email and SMS

## How to use the service

* [Loging in the user](#loging-in-the-user)
* [Using the JWT and SSID](#using-the-jwt-and-ssid)
* [Prolonging the JWT](#prolonging-the-jwt)
* [Reseting the forgotten password](#reseting-the-forgotten-password)
* [Loging out](#loging-out)
* [Reseting one time passwords](#reseting-one-time-passwords)

### Loging in the user

This process will **authenticate the user** getting their:
* Access token `JWT`
* Session `SSID`

This values will have a predefined *expired date*.


The calls below should be made via the [Login - public API](https://doc.ffc.internal/book/mw-ib/mw-gen-router-ib/router-login-public-ib/latest/index.html).

1. Call the `/getLoginScenario` to get the scenarios to follow. Each scenario is comprised of a number of pre-defined steps
2. Pick a scenario depending on the requisities eg: user and password, user and biometrics
3. Call the `/validateLoginStep` to validate each step of the scenario. This endpoint will return:
  * The next step if the process is not completed
  * The `JWT` and `SSID` values when the process was finished

![Loging in the user](loging-in-the-user.png)


### Using the JWT and SSID

`JWT` and `SSID` should be sent within the header of all requests where the API is protected (endpoints with `private` [security type](https://doc.ffc.internal/book/mw-ib/_common/urlStructure-ib.html)).

The way to build the header will be for each case:
* `JWT`. Use the **key** `Authorization` and the **value** `Bearer ` plus *JWT value*
* `SSID`. Use the **key** `ssid` and the **value** *UUID value*

Example:

`"Authorization": "Bearer 0TMfvvoz37KrI1mVVGlj2BZtnwBHyrdtJ9J3D2Irz2THqY31adk/AwVGN76jY0e3jCQOvbAKdblnTpP2OZRxFftoCF/+IFBiqbzJZWgv3ZgxrNHUwc7j4BjAwLWrhGlJYH6eiv2cmJ45GnlyCUNxIVyHpjGmN6oqeCVwtPgyZB3tXh1ywsuc2/mrnVlFnUC0WrXNg5QUm26S3XbBvihWIV/FvyUcAb/k+FrvFcH7zxUQotPmCmFQ1FMRDPJwHrxzbGG924UlskTDNSi50trpX/oD8td7tjBTpTbsIeWMb+jlJFiFgQxv3Rfuf2z0dHeStkfAELWL3VHTlKp9ihfUIC0NHUxCK9vYTFr/5a0wYZQkxacm9eXwN/FTR5+FyOKIvwmZZO1EHzlUie1VPs0qXCcfk+fXVPmawNjS3H2RO8t+HgkmxqxtSz1ADfQqEVRSE3tDrmdvyrYCOR6knAefzfoOR03SuGvmwYtNZ0Su78KZ64TsKs3RXYmp/K+Hhcjjl+QFMfLU75cYYJUlBW89alnb7sORh/e8myyJ7eOp/r4w3WQ2KkN3PiazGSHqVwwzGA4tDaJ6BYMQPZh6tu/dVlEtOAly5jc450WV0nrEjCuphBUSrnc0KBxF6XHOBF9E26WJRMP6EzjwOpgRdOegzVhlm6o/1lWPZZYuIM4VxHhDk8R9HQTflTdJaguFn1WPQyqd/PB6myZVcGB3Han2XwBpzcktG4l3D5ipFbjGIfeaOI+7MxdJzMnAWSLnQ8o8Fg2XvPHd9r/tKOpV7tWi/zHLfSuRyJpbW3kZdPcnAFx36/HRee7c4uH6MCyGZPGFUgF9hx1I/OSnrwhPepenTv4GVhwPvVj5PRypFI8/fwEsrKWs0lpNq078ljoQ6CqGYLoIyZ+0oqmMqES4BcHvfYvB2alHy6NDpoTIa9dPFTI+6XtFdtjQPEiNV/FRNkxKGLznVZU0Z3Ehl73YKp4qTCc2lw3EusWVQAdr/Xw03vfkXrRxuJEIv+PBkn7zDhIhpJOQoI9wW6Bm3OaZlCrDvTvV5LGdnV/hU7LOJeSITzq06nzPU3u8aQsulKrdSkzXk8EbPBmlPYybWoKLB4jvGQ==",
 "ssid" : "A8qPBTTa88X9yo9rh0lLacte4pg="`

### Prolonging the JWT

The `JWT` access token can be prologed with the call `/prolong` within the [Access token API](https://doc.ffc.internal/book/mw-ib/mw-gen-router-ib/router-token-private-ib/latest/).

The `JWT` access token can be prolonged until the session `SSID` is expired. When the `SSID` has expired, the User has to log in again. This `SSID` validation is done to avoid the indefinite refresh of the `JWT` access token.

### Reseting the forgotten password

1. Call `/getUnlockScenario` in [Login - public API](https://doc.ffc.internal/book/mw-ib/mw-gen-router-ib/router-login-public-ib/latest/) to get the scenarios to follow to reset the password. Pick one depending on your requirements.

2. Call `/validateUnlockStep` in [Login - public API](https://doc.ffc.internal/book/mw-ib/mw-gen-router-ib/router-login-public-ib/latest/) to validate each step of the scenario. Repeat this call until you get the `AuthenticationStepResult.result == "FINISH"`. While the `AuthenticationStepResult.result == "NEXT_STEP"`, you will get the `AuthenticationStepResult.NextStep to do.

3. Call the `\unlock` within the [Login - private API](https://doc.ffc.internal/book/mw-ib/mw-gen-router-ib/router-login-private-ib/latest/) to set the new password. The response of the request will be the new `TokensForAuthentication`.

![Loging in the user](reseting-the-forgotten-password.png)

### Logging out

Close the session with the call `/log out` within the [Logout API](https://doc.ffc.internal/book/mw-ib/mw-gen-router-ib/router-logout-private-ib/latest/index.html). This call will invalidate the JWT access token.

### Reseting one time passwords

During the [authentication process](https://doc.ffc.internal/book/mw-ib/mw-gen-user-activation-ib.html#contact-detail-verification) of the email or phone number, the user can required new values with the call `/sendOTP` in the [Resend OTP API](https://doc.ffc.internal/book/mw-ib/mw-gen-router-ib/router-otp-public-ib/latest/).

This call is preconfigurated with special protection to prevent abuse, such as reCaptcha or invocation limit.
