# Security

We have [two API security types](urlStructure-ib.md): public and private.

All public endpoints can be used by non authenticated user but all private endpoints require valid *`JWT access token`* and *`SSID`* to be sent via HTTP headers.

*`JWT access token`* is created by successful login operation. More information about *`JWT access token`* and *`SSID`* can be found in [Router](../mw-gen-router-ib.md) service description.

File download is not secured by *`JWT access token`* but specific *`JWT download token`*. The process how to work with *`JWT download token`* is described in [Document Management System](mw-gen-dms-ib.md) service.

The following error codes are returned when there are problems with authentication/authorization, see [Authorization](../../mw-ext/mw-gen-authorization-ext.html).

| Code | Name               | Reason                                                          |
|------|--------------------|-----------------------------------------------------------------|
| 401  | Unauthorized       | Non authenticated user tries to access `/api/private` endpoints.|
| 403  | Forbidden          | Authenticated user doesn't have the required rights.            |
| 405  | Method Not Allowed | Wrong method (e.g. GET, POST) used.                             |
