This service enables the management of agreements between a customer and the bank. It is used during the onboarding process (see [User Activation](mw-gen-user-activation-ib.md) service to get more information).

## Responsibilities of the Service

Service is responsible for providing the interface for:
- Creation of the new `agreement` between customer and bank during the onboarding process.
- Providing list of existing `agreement types`. An agreement type provides list of `pricing schemes` that the customer is allowed to choose from.

## Agreement definition

There can be only one agreement between the customer and the bank, even if the customer has multiple products. Agreement is not terms and conditions. The agreement stores information about `pricing scheme` selected by the customer.

## Agreement usage

During onboarding process the user is expected to chose an agreement type provided by `/agreementTypes/!list` endpoint in [Agreement API](mw-gen-agreement-ib/agreement-ib/latest/) and then to be presented with the choice between `pricing schemes` associated with the selected `agreement type`. After they have chosen the pricing scheme, it should be stored by post to `/agreements` endpoint in [Agreement API](agreement/api/).
