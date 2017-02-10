
..  index:: WOPI requests; GetPublicKeys, GetPublicKeys

..  |operation| replace:: GetPublicKeys

..  _GetPublicKeys:

GetPublicKeys
=============

..  default-domain:: http

..  post:: /wopi/ecosystem

    ..  include:: /_fragments/future_operation.rst

    ..  todo:: write more description, including expectations around key rotation

    We recommend hosts accept the anonymous access token for this operation.

    ..  include:: /_fragments/access_token_param.rst

    :code 200: Success
    :code 401: Invalid :term:`access token`
    :code 404: Resource not found/user unauthorized
    :code 500: Server error

    ..  include:: /_fragments/common_headers.rst


Response
--------

..  include:: /_fragments/json_response.rst


Required response properties
----------------------------

..  include:: /_fragments/json_response_required.rst

Modulus
    The modulus for the current public key that clients can use to verify signed data from the host. Used in
    conjunction with the *Exponent* property.

Exponent
    The exponent for the current public key that clients can use to verify signed data from the host. Used in
    conjunction with the *Modulus* property.

OldModulus
    The modulus for the previous public key that clients can use to verify signed data from the host. Used in
    conjunction with the *OldExponent* property.

OldExponent
    The exponent for the previous public key that clients can use to verify signed data from the host. Used in
    conjunction with the *OldModulus* property.


Sample response
---------------

..  literalinclude:: /_fragments/responses/GetPublicKeys.json
    :language: JSON
