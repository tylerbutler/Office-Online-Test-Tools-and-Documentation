
An optional **string** value indicating the user who owns the current lock on the file. This header may be included
when responding to the request with :http:statuscode:`409`.

If provided, this header must contain the :term:`UserFriendlyName` of the user who owns the current file lock. This
value may be displayed in the WOPI client UI.

..  important::
    Note that this header is only relevant in cases where the file is *not* locked with a WOPI lock. Locks in WOPI are
    not user-owned.
