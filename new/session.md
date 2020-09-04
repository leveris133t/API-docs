# Session

This service performs user authentication and other supporting functions.

## Responsabilities of the service

The session service is responsible for:
* **Logging the user in** to authenticate them and return their session
* **Prolonging session tokens** if required
* **Logging the user out**

## How to use the service

* [Logging the user in](#logging-the-user-in)
* [Using the JWT](#using-the-jwt)
* [Prolonging the JWT](#prolonging-the-jwt)
* [Logging out](#logging-out)

### Logging the user in

This process will **authenticate the user** and return their JWT access token with its expiration date.

To log in, follow this flow:

1. Call the endpoint `session/sessions/!getLoginSession` through the [Session - public API](session/public-api) getting the status code `418` with the `idAuthProcess` in the response. This status code notifies that a new AA flow should start.
2. Complete AA flow according to [AA](aa.md).
3. Call the endpoint `session/sessions/!getLoginSession` providing the `idAuthProcess` and using the [Session - public API](session/public-api). The endpont will return the JWT access token and its expiration date.


### Using the JWT

The JWT access token  should be sent in the header of all requests where the API is protected (endpoints with `private` [security type](url-structure.md)).

The JWT has to be added using the **key** `Authorization`and the **value** `Bearer` plus *JWT value*.

Example:

`"Authorization": "Bearer 0TMfvvoz37KrI1mVVGlj2BZtnwBHyrdtJ9J3D2Irz2THqY31adk/AwVGN76jY0e3jCQOvbAKdblnTpP2OZRxFftoCF/+IFBiqbzJZWgv3ZgxrNHUwc7j4BjAwLWrhGlJYH6eiv2cmJ45GnlyCUNxIVyHpjGmN6oqeCVwtPgyZB3tXh1ywsuc2/mrnVlFnUC0WrXNg5QUm26S3XbBvihWIV/FvyUcAb/k+FrvFcH7zxUQotPmCmFQ1FMRDPJwHrxzbGG924UlskTDNSi50trpX/oD8td7tjBTpTbsIeWMb+jlJFiFgQxv3Rfuf2z0dHeStkfAELWL3VHTlKp9ihfUIC0NHUxCK9vYTFr/5a0wYZQkxacm9eXwN/FTR5+FyOKIvwmZZO1EHzlUie1VPs0qXCcfk+fXVPmawNjS3H2RO8t+HgkmxqxtSz1ADfQqEVRSE3tDrmdvyrYCOR6knAefzfoOR03SuGvmwYtNZ0Su78KZ64TsKs3RXYmp/K+Hhcjjl+QFMfLU75cYYJUlBW89alnb7sORh/e8myyJ7eOp/r4w3WQ2KkN3PiazGSHqVwwzGA4tDaJ6BYMQPZh6tu/dVlEtOAly5jc450WV0nrEjCuphBUSrnc0KBxF6XHOBF9E26WJRMP6EzjwOpgRdOegzVhlm6o/1lWPZZYuIM4VxHhDk8R9HQTflTdJaguFn1WPQyqd/PB6myZVcGB3Han2XwBpzcktG4l3D5ipFbjGIfeaOI+7MxdJzMnAWSLnQ8o8Fg2XvPHd9r/tKOpV7tWi/zHLfSuRyJpbW3kZdPcnAFx36/HRee7c4uH6MCyGZPGFUgF9hx1I/OSnrwhPepenTv4GVhwPvVj5PRypFI8/fwEsrKWs0lpNq078ljoQ6CqGYLoIyZ+0oqmMqES4BcHvfYvB2alHy6NDpoTIa9dPFTI+6XtFdtjQPEiNV/FRNkxKGLznVZU0Z3Ehl73YKp4qTCc2lw3EusWVQAdr/Xw03vfkXrRxuJEIv+PBkn7zDhIhpJOQoI9wW6Bm3OaZlCrDvTvV5LGdnV/hU7LOJeSITzq06nzPU3u8aQsulKrdSkzXk8EbPBmlPYybWoKLB4jvGQ=="`

### Prolonging the JWT

The JWT access token can be prolonged by calling `/session/sessions/!prolong` through the [Session - private API](session/private-api) returning a new `JWT` token.

### Logging out

Close the session calling the endpoint `/session/sessions/!logout` through the [Session - private API](session/private-api). This call will invalidate the JWT access token.
