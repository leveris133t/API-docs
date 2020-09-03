# The Basics

Our API is based on [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) principles with small exceptions and without [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) support.

Request and response payload is always in [JSON](https://www.json.org/) format.

There is also websocket channel for asynchronous communication between server and front-end application. It is described in [Asynchronous Communication](../mw-gen-asynccomm-ib.md) service.

We are using [RAML 1.0](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md/) for our API description.

RAML files contains all information required for integration. In fact our server and client API implementation is fully generated from RAML files.

