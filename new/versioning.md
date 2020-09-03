# Versioning

## Version number semantics

API version number can be found in the Summary section of the API documentation.

It consists of two numbers separated by dot. The first number is called major number and the second one is the minor number.

The major number is increased by one whenever one or more backward incompatible changes are made in the API. The minor number is reset to zero in such case.

When only backward compatible changes are made in the API then the major number remains the same and the minor number is increased by one.

## Backward compatible changes

These changes can be made without previous notification.

All client applications needs to be written in such way that these kind of changes will not break their functionality.

Backward compatible changes are:

* adding of new API endpoint
* adding of new **optional** parameter to the existing endpoint (excluding update operation)
* adding of new enumeration value for an attribute in the **request** (not in response)
* documentation modification

All other modifications are considered as backward incompatible.
