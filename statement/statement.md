# Statement

The **statement service** is responsible for configuring and generating a summary of all financial transactions which have occurred on a user's account for a given period of time.

## Responsibilities of the service

The statement service is responsible for:

* **Configuring the regular statement generation** e.g. activation, frequency, delivery channel
* **Providing regular statement history**
* **Generating on-demand statements**

## Glossary

* **Regular statement**: Statements which are generated on a periodic basis. The generation is triggered by the system on a `Monthly`, `Quaterly` or `Yearly` basis.
* **On-demand statement**: Statements that are generated on request by the user, for the given time period e.g. the user wants a Statement from *May 8, 2019* to *May 27, 2019* etc.

## How to use the service

The statement service requires that the user has been:
* **Onboarded** and is
* **Logged in** to the Internet Banking service

All endpoints for this service are available within the [Statement API](https://doc.ffc.internal/book/mw-ib/mw-gen-statement-ib/statement-ib/latest/index.html).

* [Configuring the regular statement generation](#configuring-the-regular-statement-generation)
* [Providing the regular statement history](#providing-the-regular-statement-history)
* [Generating on-demand statements](#generating-on-demand-statements)


### Configuring the regular statement generation

Each account has a statement **configuration**, where attributes such as `Frequency` and `Delivery Channel` can be configured. These attributes determine:
 * **When** the regular statement will be generated and
 * **Where** the regular statement will be sent

The user can also turn off regular statments altogether.
 
Use **POST** `/settings/!list` to get the full list of statement settings.

Call **GET** `/settings/{id}` to get full setting detail or use **PATCH** `/settings/{id}` to update it.

### Providing the regular statement history

All the regular statements are kept in history. A user can retrieve and download them as many times as they wish.

Use the call **POST** `/statement/{list}` to get the full list of regular statements. A statement can downloaded using the [Document Management System API](https://doc.ffc.internal/book/mw-ib/mw-gen-dms-ib.html).

### Generating on-demand statements

A user can generate a statement for a given time period by calling the  **POST** `/statement/!generate` endpoint.

On-demand statements are saved to the user statements history.
