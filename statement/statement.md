# Statement

The **statement service** is responsible for configuring and generating a summary of the financial transactions which have occurred in the user's bank account within a period of time.

## Responsibilities of the service

The statement service is responsible for:

* **Configuring the regular statement generation** e.g. activation, frequency, delivery channel
* **Providing the regular statement history**
* **Generating on-demand statements**

## Glossary

* **Regular statement**: Statement which is generated on a periodic basis. The generation is triggered by the system according to a pre-configured frequency
* **On-demand statement**: Statement which is generated according to the request of a user based on a time interval e.g. Statement from *May 8, 2019* to *May 27, 2019*

## How to use the service

The statement service requires that the user has been:
* **Onboarded** and is
* **Logged in** to the Internet Banking service

All endpoints for this service are available within the [Statement API](https://doc.ffc.internal/book/mw-ib/mw-gen-statement-ib/statement-ib/latest/index.html).

* [Configuring the regular statement generation](#configuring-the-regular-statement-generation)
* [Providing the regular statement history](#providing-the-regular-statement-history)
* [Generating on-demand statements](#generating-on-demand-statements)


### Configuring the regular statement generation

Each account has a **default configuration** for the regular statement based on attributes such as frequency or delivery channel. These attributes will indicate:
 * **When** the regular statement will be triggered and
 * **Where** the regular statement will be sent

Call **GET** `/settings/{id}` to get the configuration of the regular statement or use **PATCH** `/settings/{id}` to update it.

Use **POST** `/settings/!list` to get the full list of regular statements of a user based on a filter.

### Providing the regular statement history

All the regular statement generated will be kept in a history. The user can get all of them and download them as many times as he wants.

Use the call **POST** `/statement/{list}` to get the full list of generated regular statements. If you want to download a file of the previous list, take a look at the [Document Management System](https://doc.ffc.internal/book/mw-ib/mw-gen-dms-ib.html).


### Generating on-demand statements

Use the call **POST** `/statement/!generate` to request the generation of a statement within a time interval. This statement can be downloaded from the client's device.

The on-demand statements will not be saved on a history.
