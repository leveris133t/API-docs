# URL Structure

The endpoint URI is composed of segments separated by the slash (`/`) character in the following order.

* **Base URI** - it is the same for all operations in one API. It consists of

    * **protocol**, **hostname**, **portnumber** - it depends according to environment you are connecting to
    * **prefix** (optional) - it depends according to environment you are connecting to
    * **service code**
    * the word `api`
    * **security type**, which is one of

        * `private` - secured API. This API is not accessible for non-logged users. JWT token needs to be used for such APIs.
        * `public` - no access control is performed
* **API name**
* **Collection name** (referred as a resource in the REST description)
* **Entity ID** (optional; referred as a resource ID in the REST description) - this part is used only when working with one entity instance (not used for collections)
* **Custom verb** (optional) - the name of the operation carried out on an entity (or collection) if it is not one of four standard operations (Get, Create, Update or Delete). A custom verb must be preceded by an exclamation mark (`!`).

Collection and Entity ID can repeat (we are using hierarchical structure for API entities). The Entity ID between two collection names cannot be omitted (only the last one is optional).

For example the URI
`https://example.org/ib/api/mw-gen-user-activation-ib/api/public/user-activation/processes/123456/auth-processes/987654/!validateCurrentAuthSubprocessStep`
consists of

* Base URI - `https://example.org/ib/api/mw-gen-user-activation-ib/api/public`, which consists of
    * protocol, hostname, portnumber - `https://example.org`
    * prefix - `/ib/api`
    * service code - `mw-gen-user-activation-ib`
    * the word - `api`
    * security type - `public`
* API name - `user-activation`
* Collection - `processes`
* Entity ID - `123456`
* Collection - `auth-processes`
* Entity ID - `987654`
* Custom verb - `!validateCurrentAuthSubprocessStep`
