
..  _PostMessage:

Using PostMessage to interact with the |wac| application iframe
===============================================================

..  spelling::

    msg
    ui
    wdUserSession


..  default-domain:: js

You can integrate your own UI into Office Online applications. This way, you can use your UI for actions on Office
documents, such as sharing.

To integrate with Office Online in this way, implement the
`HTML5 Web Messaging protocol <http://www.w3.org/TR/webmessaging/>`_. The Web Messaging protocol,
also known as PostMessage, allows the |wac| frame to communicate with its parent :term:`host page`, and
vice-versa. The following example shows the general syntax for PostMessage. In this example, ``otherWindow`` is a
reference to another window that ``msg`` will be posted to.

..  function:: otherWindow.postMessage(msg, targetOrigin)

    :param string msg: A string (or JSON object) that contains the message
        data.
    :param string targetOrigin: Specifies what the origin of ``otherWindow`` must be for the event to be dispatched.
        This value will be set to the :term:`PostMessageOrigin` property provided in :ref:`CheckFileInfo`. The literal
        string ``*``, while supported in the PostMessage protocol, is not allowed by Office Online.

Message format
--------------

All messages posted to and from the Office Online application frame are posted using the
:func:`~otherWindow.postMessage` function. Each message (the ``msg`` parameter in the
:func:`~otherWindow.postMessage` function) is a JSON-formatted object of the form:

..  data:: message

    **MessageId** *(string)*
        The name of the message being posted.
    **SendTime** *(long)*
        The time the message was sent, expressed as milliseconds since
        midnight 1 January 1970 UTC.

        ..  tip:: You can get this value in most modern browsers using the ``Date.now()`` method in JavaScript.

    **Values** *(JSON-formatted object)*
        The data associated with the message. This varies per message.

The following example shows the msg parameter for the :data:`Host_PerfTiming` message.

..  code-block:: JSON

    {
        "MessageId": "Host_PerfTiming",
        "SendTime": 1329014075000,
        "Values": {
            "Click": 1329014074800,
            "Iframe": 1329014074950,
            "HostFrameFetchStart": 1329014074970,
            "RedirectCount": 1
        }
    }

Sending messages to the Office Online iframe
--------------------------------------------

To send messages to the Office Online iframe, you must set the :term:`PostMessageOrigin` property in your WOPI
:ref:`CheckFileInfo` response to the URL of your host page. If you do not do this, Office Online will ignore any
messages you send to its iframe.

You can send the following messages; all others are ignored:

* :data:`App_PopState`
* :data:`Blur_Focus`
* :data:`Grab_Focus`
* :data:`Host_InsertImage`
* :data:`Host_PerfTiming`
* :data:`Host_PostmessageReady`

..  data:: App_PopState

    ..  include:: /_fragments/onenote_only.rst

    The App_PopState message signals the Office Online application that state has been popped from the HTML5 History
    API to which the application should navigate to using the URL. This message should be triggered from an
    `onpopstate` listener in the host page.

    ..  attribute:: Values
        :noindex:

        Url *(string)*
            The URL associated with the popped history state.

        State *(JSON-formatted object)*
            The data associated with the state.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "App_PopState",
            "SendTime": 1329014075000,
            "Values": {
                "Url": "https://www.contoso.com/abc123/contents?wdtarget=pagexyz",
                "State": {
                    "Value": 0
                }
            }
        }

..  data:: Blur_Focus

    The Blur_Focus message signals the Office Online application to stop aggressively grabbing focus. Hosts should
    send this message whenever the host application UI is drawn over the Office Online frame, so that the Office
    application does not interfere with the UI behavior of the host.

    This message only affects Office Online edit modes; it does not affect view modes.

    ..  tip::
        When the host application displays UI over Office Online, it should put a full-screen dimming effect over the
        Office Online UI, so that it is clear that the Office application is not interactive.

    ..  attribute:: Values
        :noindex:

        *Empty.*

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "Blur_Focus",
            "SendTime": 1329014075000,
            "Values": { }
        }

..  data:: Grab_Focus

    The Grab_Focus message signals the Office Online application to resume aggressively grabbing focus. Hosts should
    send this message whenever the host application UI that is drawn over the Office Online frame is closing. This
    allows the Office application to resume functioning.

    This message only affects Office Online edit modes; it does not affect view modes.

    ..  attribute:: Values
        :noindex:

        *Empty.*

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "Grab_Focus",
            "SendTime": 1329014075000,
            "Values": { }
        }

..  data:: Host_InsertImage

    The Host_InsertImage message should be sent to the |wac| iframe when the user selects an image from the host file
    selection UI. It is closely related to the :js:data:`UI_InsertImage` message.

    ..  attribute:: Values
        :noindex:

        Hosts can support inserting multiple images at once. Thus, the *Values* property should be an array of
        objects, where each object in the array has the following properties:

        **Extension** *(string)*
            The file extension for the selected file. This value must begin with a ``.``.

        **SourceUrl** *(string)*
            A URL that the WOPI client can use to retrieve the selected file.

        **Width** *(integer)*
            The width of the image in pixels.

        **Height** *(integer)*
            The height of the image in pixels.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "Host_InsertImage",
            "SendTime": 1329014075000,
            "Values": [
                {
                    "Extension": ".jpg",
                    "SourceUrl": "https://contosodrive.com/images/1aed278abdec74ab.jpg",
                    "Width": 800,
                    "Height": 600
                },
                {
                    "Extension": ".jpg",
                    "SourceUrl": "https://contosodrive.com/images/259aef4d21a84edb.jpg",
                    "Width": 800,
                    "Height": 600
                }
            ]
        }

..  data:: Host_PerfTiming

    Provides performance related timestamps from the host page. Hosts should send this message when the Office
    Online frame is created so load performance can be more accurately tracked.

    ..  attribute:: Values
        :noindex:

        **Click** *(integer)*
            The timestamp, in ticks, when the user selected a link that launched the Office Online application. For
            example, if the host exposed a link in its UI that launches an Office Online application, this timestamp
            is the time the user originally selected that link.

        **Iframe** *(integer)*
            The timestamp, in ticks, when the host created the Office Online iframe when the user selected the link.

        **HostFrameFetchStart** *(integer)*
            The result of the `PerformanceTiming.fetchStart`_ attribute, if the browser supports the
            `W3C NavigationTiming API`_. If the NavigationTiming API is not supported by the browser, this must be 0.

        **RedirectCount** *(integer)*
            The result of the `PerformanceNavigation.redirectCount`_ attribute, if the browser supports the
            `W3C NavigationTiming API`_. If the NavigationTiming API is not supported by the browser, this must be 0.

.. _W3C NavigationTiming API: http://www.w3.org/TR/navigation-timing/
.. _PerformanceTiming.fetchStart: http://www.w3.org/TR/navigation-timing/#dom-performancetiming-fetchstart
.. _PerformanceNavigation.redirectCount: http://www.w3.org/TR/navigation-timing/#dom-performancenavigation-redirectcount

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "Host_PerfTiming",
            "SendTime": 1329014075000,
            "Values": {
                "Click": 1329014074800,
                "Iframe": 1329014074950,
                "HostFrameFetchStart": 1329014074970,
                "RedirectCount": 1
            }
        }

..  data:: Host_PostmessageReady

    Office Online delay-loads much of its JavaScript code, including most of its PostMessage senders and listeners.
    You might choose to follow this pattern in your WOPI host page. This means that your outer host page and the
    Office Online iframe must coordinate to ensure that each is ready to receive and respond to messages.

    To enable this coordination, Office Online sends the :data:`App_LoadingStatus` message only after all of its message
    senders and listeners are available. In addition, Office Online listens for the :data:`Host_PostmessageReady`
    message from the outer frame. Until it receives this message, some UI, such as the :guilabel:`Share` button, is
    disabled.

    Until your host page receives the :data:`App_LoadingStatus` message, the Office Online frame cannot respond to any
    incoming messages except :data:`Host_PostmessageReady`. Office Online does not delay-load its
    :data:`Host_PostmessageReady` listener; it is available almost immediately upon iframe load.

    If you are delay-loading your PostMessage code, you must ensure that your :data:`App_LoadingStatus` listener is not
    delay-loaded. This will ensure that you can receive the :data:`App_LoadingStatus` message even if your other
    PostMessage code has not yet loaded.

    The following is the typical flow:

    1. Host page begins loading.
    2. Office Online frame begins loading. Some UI elements are disabled, because :data:`Host_PostmessageReady` has
       not yet been sent by the host page.
    3. Host page finishes loading and sends :data:`Host_PostmessageReady`. No other messages are sent because the
       host page hasn't received the :data:`App_LoadingStatus` message from the Office Online frame.
    4. Office Online frame receives :data:`Host_PostmessageReady`.
    5. Office Online frame finishes loading and sends :data:`App_LoadingStatus` to host page.
    6. Host page and Office Online communicate by using other PostMessage messages.

    ..  attribute:: Values
        :noindex:

        *Empty.*

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "Host_PostmessageReady",
            "SendTime": 1329014075000,
            "Values": { }
        }


Listening to messages from the Office Online iframe
---------------------------------------------------

The Office Online iframe will send messages to the host page. On the receiving end, the host page will receive a
MessageEvent. The origin property of the MessageEvent is the origin of the message, and the data property is the
message being sent. The following code example shows how you might consume a message.

.. code-block:: javascript

    function handlePostMessage(e) {
        // The actual message is contained in the data property of the event.
        var msg = JSON.parse(e.data);

        // The message ID is now a property of the message object.
        var msgId = msg.MessageId;

        // The message parameters themselves are in the Values
        // parameter on the message object.
        var msgData = msg.Values;

        // Do something with the message here.
    }
    window.addEventListener('message', handlePostMessage, false);

The host page receives the following messages; all others are ignored:

* :data:`App_LoadingStatus`
* :data:`App_PushState`
* :data:`Edit_Notification`
* :data:`File_Rename`
* :data:`UI_Close`
* :data:`UI_Edit`
* :data:`UI_Sharing`
* :data:`UI_Workflow`


..  _outgoing postmessage common values:

Common Values
~~~~~~~~~~~~~

In addition to message-specific values passed with each message, Office Online sends the following common values with
every outgoing PostMessage:

..  glossary::
    :sorted:

    ui-language *(string)*
        The LCID of the language Office Online was loaded in. This value will not match the value provided using the
        :term:`UI_LLCC` placeholder. Instead, this value will be the numeric LCID value (as a *string*) that
        corresponds to the language used. See :ref:`languages` for more information.

        This value may be needed in the event that Office Online renders using a language different than the one
        requested by the host, which may occur if Office Online is not localized in the language requested. In that
        case, the host may choose to draw its own UI in the same language that Office Online used.

    wdUserSession *(string)*
        The ID of the Office Online session. This value can be logged by host and used when
        :ref:`troubleshooting <troubleshooting>` issues with Office Online. See :ref:`session id` for more
        information about this value.


..  data:: App_LoadingStatus

    The App_LoadingStatus message is posted after the Office Online application frame has loaded. Until the host
    receives this message, it must assume that the Office Online frame cannot react to any incoming messages except
    :data:`Host_PostmessageReady`.

    ..  attribute:: Values
        :noindex:

        DocumentLoadedTime *(long)*
            The time that the frame was loaded.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "App_LoadingStatus",
            "SendTime": 1329014075000,
            "Values": {
                "DocumentLoadedTime": 1329014074983,
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: App_PushState

    ..  include:: /_fragments/onenote_only.rst

    The App_PushState message is posted when the user changes the state of Office Online application in a way
    which the user may wish to return to later, requesting to capture it in the HTML 5 History API. In receiving
    this message, the Host page should using `history.pushState` to capture the state for a potential later
    state pop.

    To send this message, the :term:`AppStateHistoryPostMessage` property in the :ref:`CheckFileInfo` response
    from the host must be set to ``true``. Otherwise Office Online will not send this message.

    ..  attribute:: Values
        :noindex:

        Url *(string)*
            The URL associated with the message.

        State *(JSON-formatted object)*
            The data associated with the state.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "App_PushState",
            "SendTime": 1329014075000,
            "Values": {
                "Url": "https://www.contoso.com/abc123/contents?wdtarget=pagexyz",
                "State": {
                    "Value": 0
                },
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: Edit_Notification

    The Edit_Notification message is posted when the user first makes an edit to a document, and every five minutes
    thereafter, if the user has made edits in the last five minutes. Hosts can use this message to gauge whether
    users are interacting with Office Online. In coauthoring sessions, hosts cannot use the WOPI calls for
    this purpose.

    To send this message, the :term:`EditNotificationPostMessage` property in the :ref:`CheckFileInfo` response from
    the host must be set to ``true``. Otherwise Office Online will not send this message.

    ..  attribute:: Values
        :noindex:

        :ref:`Common values <outgoing postmessage common values>` only.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "Edit_Notification",
            "SendTime": 1329014075000,
            "Values": {
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: File_Rename

    The File_Rename message is posted when the user renames the current file in Office Online. The host can use this
    message to optionally update the UI, such as the title of the page.

    ..  note::
        If the host does not return the :term:`SupportsRename` parameter in their :ref:`CheckFileInfo` response, then
        the rename UI will not be available in Office Online.

    ..  attribute:: Values
        :noindex:

        NewName *(string)*
            The new name of the file.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "File_Rename",
            "SendTime": 1329014075000,
            "Values": {
                "NewName": "Renamed Document",
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: UI_Close

    The UI_Close message is posted when the Office Online application is closing, either due to an error or a user
    action. Typically, the URL specified in the :term:`CloseUrl` property in the :ref:`CheckFileInfo` response is
    displayed. However, hosts can intercept this message instead and navigate in an appropriate way.

    To send this message, the :term:`ClosePostMessage` property in the :ref:`CheckFileInfo` response from the host
    must be set to ``true``. Otherwise Office Online will not send this message.

    ..  attribute:: Values
        :noindex:

        :ref:`Common values <outgoing postmessage common values>` only.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "UI_Close",
            "SendTime": 1329014075000,
            "Values": {
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: UI_Edit

    The UI_Edit message is posted when the user activates the :guilabel:`Edit` UI in Office Online. This UI is only
    visible when using the :wopi:action:`view` action.

    To send this message, the :term:`EditModePostMessage` property in the :ref:`CheckFileInfo` response from the host
    must be set to ``true``. Otherwise Office Online will not send this message and will redirect the inner iframe to
    an edit action URL instead.

    Hosts may choose to use this message in cases where they want more control over the user's transition to edit
    mode. For example, a host may wish to prompt the user for some additional host-specific information before
    navigating.

    ..  attribute:: Values
        :noindex:

        :ref:`Common values <outgoing postmessage common values>` only.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "UI_Edit",
            "SendTime": 1329014075000,
            "Values": {
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: UI_FileVersions

    The UI_FileVersions message is posted when the user activates the :guilabel:`Previous Versions` UI in |wac|. The
    host should use this message to trigger any custom file version history UI.

    To send this message, the :term:`FileVersionPostMessage` property in the :ref:`CheckFileInfo` response from the
    host must be set to ``true``. Otherwise |wac| will not send this message.

    ..  attribute:: Values
        :noindex:

        :ref:`Common values <outgoing postmessage common values>` only.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "UI_FileVersions",
            "SendTime": 1329014075000,
            "Values": {
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: UI_InsertImage

    The UI_InsertImage message is posted when the user activates the *Insert Image* UI in Office Online. This message
    can be used to allow users to insert images that are stored in the host's storage. Hosts are expected to display
    a modal file selection UI in response to this message. After the user has selected an image, the host sends the
    :js:data:`Host_InsertImage`` message to the |wac| frame.

    To send this message, the :term:`InsertOnlinePicturePostMessage` property in the :ref:`CheckFileInfo` response
    from the host must be set to ``true``. Otherwise Office Online will not send this message.

    ..  attribute:: Values
        :noindex:

        FileExtensionFilterList *(array of strings)*
            An array of strings representing the file extensions that the host should filter their file selection UI.
            All extensions must begin with a ``.``.


    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "UI_InsertImage",
            "SendTime": 1329014075000,
            "Values": {
                "FileExtensionFilterList": [".jpg", ".jpeg", ".png", ".gif"],
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: UI_Sharing

    The UI_Sharing message is posted when the user activates the :guilabel:`Share` UI in Office Online. The host should
    use this message to trigger any custom sharing UI.

    To send this message, the :term:`FileSharingPostMessage` property in the :ref:`CheckFileInfo` response from the
    host must be set to ``true``. Otherwise Office Online will not send this message.

    ..  attribute:: Values
        :noindex:

        :ref:`Common values <outgoing postmessage common values>` only.

    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "UI_Sharing",
            "SendTime": 1329014075000,
            "Values": {
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }

..  data:: UI_Workflow

    The UI_Workflow message is posted when the user activates the :guilabel:`Workflow` UI in Office Online. The host
    should use this message to trigger any custom workflow UI.

    To send this message, the :term:`WorkflowPostMessage` property in the :ref:`CheckFileInfo` response from the
    host must be set to ``true``. Otherwise Office Online will not send this message.

    ..  attribute:: Values
        :noindex:

        WorkflowType *(string)*
            The :term:`WorkflowType` associated with the message. This will match one of the values provided by the
            host in the :term:`WorkflowType` property in :ref:`CheckFileInfo`.


    ..  rubric:: Example Message:

    ..  code-block:: JSON

        {
            "MessageId": "UI_Workflow",
            "SendTime": 1329014075000,
            "Values": {
                "WorkflowType": "Submit",
                "wdUserSession": "3692f636-2add-4b64-8180-42e9411c4984",
                "ui-language": "1033"
            }
        }
