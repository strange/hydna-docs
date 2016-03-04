Hydna2 documentation
====================

Manifest
--------

The *manifest* is a JSON document responsible for routing events to *event
handlers*.

Example (current syntax)::

    {
        "channels": [
            {
                "path": "/chat",
                "source": "function onopen(event) { event.allow(); }"
            },
            {
                "path": "/ticker",
                "file": "ticker.js"
            }
        ],
        "ticks": [
            {
                "path": "/ticker",
                "interval": 5000,
                "identifier": "test-tick"
            }
        ]
    }

Example (proposed new syntax)::

    {
        "handlers": [
            {
                "path": "/chat",
                "source": "function onopen(event) { event.allow(); }"
            },
            {
                "path": "/ticker",
                "file": "ticker.js"
            }
        ],
        "agents": [
            {
                "type": "tick",
                "path": "/ticker",
                "settings": {
                    "interval": 5000,
                    "identifier": "test-tick"
                }
            }
        ]
    }


Event handlers
--------------

``onopen(event)``
~~~~~~~~~~~~~~~~~

Triggered when a client attempts to connect to a path. The event object
contains the following:

=============== =============================================================
Attribute       Description
=============== =============================================================
``domain``      Name of the current domain (string)
``bindings``    Any **bindings** extracted from the path (object)
``path``        The current path (string)
``querystring`` The raw querystring (string)
``transport``   Name of the transport (``http`` or ``ws``) (string)
``secure``      Booleand dictating whether the connection is encrypted
                (boolean)
``allow()``     Allow the client to open the path (function)
``deny()``      Deny the request to open the path (function)
=============== =============================================================

Paths that do not link to a behavior that defines a `onopen`-handler will
automatically allow connections. Paths that do define the handler will
**deny** all requests unless exclicitly allowed with a call to
``event.allow()``.

Example (current syntax)::

    function onopen(event) {
        event.allow();
    }


``onclose(event)``
~~~~~~~~~~~~~~~~~~

Triggered when a client closes a path or is disconnected. The event object
contains the following:

=============== =============================================================
Attribute       Description
=============== =============================================================
``domain``      Name of the current domain (string)
``bindings``    Any **bindings** extracted from the path (object)
``path``        The current path (string)
``reason``      An optional reason for the event (string)
=============== =============================================================

Example (current syntax)::

    function onclose(event) {
        console.log(event);
    }


``ondata(event)``
~~~~~~~~~~~~~~~~~~

Triggered when a client sends data to a path. The event object contains the
following:

=============== =============================================================
Attribute       Description
=============== =============================================================
``domain``      Name of the current domain (string)
``bindings``    Any **bindings** extracted from the path (object)
``path``        The current path (string)
``querystring`` The raw querystring (string)
``transport``   Name of the transport (``http`` or ``ws``) (string)
``secure``      Booleand dictating whether the connection is encrypted
                (boolean)
``data``        The data sent (string)
``relay()``     Automaically relay the data to all clients connected to
                the path (function)
=============== =============================================================

Paths that do not link to a behavior that defines a `ondata`-handler will
automatically relay the data to all connected clients. Data sent to paths that
do define the handler will not be sent unless it is explicitly sent with
either `event.relay()` or `Channel.send(path, msg)`.

Example (current syntax)::

    function ondata(event) {
        Channel.send(event.path, event.data);
    }

Or the same effect but more efficient::

    function ondata(event) {
        event.relay();
    }


``onrequest(request)``
~~~~~~~~~~~~~~~~~~~~~~

Triggered when a HTTP request is made.


``onevent(request)``
~~~~~~~~~~~~~~~~~~~~~~


API
---

``Channel``
~~~~~~~~~~~

An api to interact with Hydna channels. All functions in this module return a
``Promise``-instance.

``Channel.send(path, data)``
````````````````````````````

Send data to a channel.

Example::

    Channel.send('/world', 'Hello!');


``Http``
~~~~~~~~

An api to make HTTP requests. All functions in this module return a
``Promise``-instance.

``Http.get(url, [options])``
````````````````````````````

Make a HTTP GET request to ``url``.

Example::

    Http.get('http://httpbin.org/get?test=fest', {
        headers: { 'Content-Type': 'application/json' },
        payload: {test: 'fest'}
    }).then(function(resp) {
        console.log(resp);
    }).catch(function(error) {
        console.log(error);
    });


``Http.post(url, [options])``
`````````````````````````````

Make a HTTP POST request to ``url``.

Example::

    Http.post('http://httpbin.org/post', {
        payload : 'hello'
    }).then(function(data) {
        console.log(data);
    }).catch(function(error) {
        console.log(error);
    });


``Http.put(url, [options])``
````````````````````````````

Make a HTTP PUT request to ``url``.

Example::

    Http.put('http://httpbin.org/put', {
        payload : 'hello'
    }).then(function(data) {
        console.log(data);
    }).catch(function(error) {
        console.log(error);
    });


``Http.delete(url, [options])``
```````````````````````````````

Make a HTTP DELETE request to ``url``.

Example::

    Http.delete('http://httpbin.org/delete', {
        payload : 'hello'
    }).then(function(data) {
        console.log(data);
    }).catch(function(error) {
        console.log(error);
    });
