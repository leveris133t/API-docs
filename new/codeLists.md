# Code list API

Each service has one additional API which can be used to obtain very simple code list (typically for comboboxes on front end UI).

When you want to get the content of the code list then you need to know the code list name. You can found it in the description of some attributes in API documentation.

Then you need to make GET request on this endpoint: `{environment URI}/{service code}/api/public/codelist/codelists/{code list name}`

The meaning of placeholders:
* `environment URI` contains protocol, hostname, port number and prefix. See [URL Structure](urlStructure-ib) for more details.
* `service code` - the same service code which is used by the API where you have found the code list name

The response contains the name of the code list and list of simple objects with `code` and `description` attributes.
The `code` is the value which is used by attributes in other endpoints and the `description` contains english text - the textual representation of the value.

For example if you send this request:

```shell
$ curl --request GET \
  --url http://example.com/ib/api/mw-jumio-ib/api/public/codelist/codelists/jumioPoaDocumentType \
  --header 'accept: application/json' \
  --header 'content-type: application/json'
```

then you will get this response:

```json
{
    "code": "jumioPoaDocumentType",
    "data": [
        {
            "code": "BS",
            "description": "Bank statement"
        },
        {
            "code": "UB",
            "description": "Utility bill"
        },
        {
            "code": "PB",
            "description": "Phone bill"
        }
    ]
}
```

You can also get list of all code lists from one service via POST `{environment URI}/{service code}/api/public/codelist/codelists/!list`.

The request payload is empty and the response is the array of code list definitions.

There is an example of such request:

```shell
$ curl --request POST \
  --url 'http://example.com/ib/api/mw-jumio-ib/api/public/codelist/codelists/!list' \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --data '{}'
```

and response

```json
[
    {
        "code": "jumioPoaDocumentType",
        "data": [
            {
                "code": "BS",
                "description": "Bank statement"
            },
            {
                "code": "UB",
                "description": "Utility bill"
            },
            {
                "code": "PB",
                "description": "Phone bill"
            }
        ]
    }
]
```

*The code list concept described on this page is deprecated but still used by some APIs. We are moving these code lists to be a part of standard application API, documented in RAML files and with localization support.*