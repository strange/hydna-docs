Hydna2 documentation
====================

Transports
----------

Hydna supports the following transports:

- HTTP
- WebSocket
- EventSource


HTTP
~~~~

Send POST, GET, PUT or DELETE requests::

    curl -d test=fest http://127.0.0.1:8070/test/

Triggers the following handlers:

- ``onrequest``


WebSocket
~~~~~~~~~

Connect with WebSocket::

    var ws = new WebSocket("ws://127.0.0.1:8070/test/");

    ws.onmessage = function(event) {
        console.log(event);
    };

Triggers the following handlers:

- ``onopen``
- ``onmessage``
- ``onclose``


EventSource
~~~~~~~~~~~

Connect with EventSource::

    var es = new EventSource("http://127.0.0.1:8070/test/");

    es.onmessage = function(event) {
        console.log(event);
    };

Triggers the following handlers:

- ``onopen``
- ``onclose``


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
``ref``         Unique reference of the connected client (string)
``ip``          IP address of the connected client (string)
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
``ref``         Unique reference of the connected client (string)
``bindings``    Any **bindings** extracted from the path (object)
``path``        The current path (string)
``reason``      An optional reason for the event (string)
=============== =============================================================

Example (current syntax)::

    function onclose(event) {
        console.log(event);
    }


``onmessage(event)``
~~~~~~~~~~~~~~~~~~

Triggered when a client sends data to a path. The event object contains the
following:

=============== =============================================================
Attribute       Description
=============== =============================================================
``domain``      Name of the current domain (string)
``ref``         Unique reference of the connected client (string)
``ip``          IP address of the connected client (string)
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

Paths that do not link to a behavior that defines a `onmessage`-handler will
automatically relay the data to all connected clients. Data sent to paths that
do define the handler will not be sent unless it is explicitly sent with
either `event.relay()` or `Channel.send(path, msg)`.

Example (current syntax)::

    function onmessage(event) {
        Channel.send(event.path, event.data);
    }

Or the same effect but more efficient::

    function onmessage(event) {
        event.relay();
    }


``onrequest(request)``
~~~~~~~~~~~~~~~~~~~~~~

Triggered when a HTTP request is made.

=============== =============================================================
Attribute       Description
=============== =============================================================
``domain``      Name of the current domain (string)
``ref``         Unique reference of the connected client (string)
``ip``          IP address of the connected client (string)
``bindings``    Any **bindings** extracted from the path (object)
``path``        The current path (string)
``querystring`` The raw querystring (string)
``transport``   Name of the transport (``http`` or ``ws``) (string)
``secure``      Booleand dictating whether the connection is encrypted
                (boolean)
``data``        The data sent (string)
``resp(body)``  Respond to the request.
=============== =============================================================

Example::

    function onrequest(req) {
        req.resp("Hello world");
    }


``onevent(request)``
~~~~~~~~~~~~~~~~~~~~~~

Triggered when an event is dispatched on a path.

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
``resp()``      Respond to the request.
=============== =============================================================

Agents
------

**Agents** are long-lived processes that trigger events on paths.

``Tick agent``
~~~~~~~~~~~~~~

The tick-agent triggers an event on a path on interval. The event object has
the following attributes:

=============== =============================================================
Attribute       Description
=============== =============================================================
``domain``      Name of the current domain (string)
``bindings``    Any **bindings** extracted from the path (object)
``path``        The current path (string)
``querystring`` The raw querystring (string)
``type``        Type of agent ('tick')
``identifier``  Unique identifier of the agent (string)
``interval``    Interval at which the agent is running
=============== =============================================================

Example handler::

    function onevent(event) {
        const url = 'http://quotes.stormconsultancy.co.uk/random.json';
        http.get(url).then(function(resp) {
            const quote = JSON.parse(resp.body);
            const msg = `[quote] "${quote.quote}" by ${quote.author}\n`;
            Channel.send(event.path, msg);
        });
    }


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


``Client``
~~~~~~~~~~

Rename to "Connection"?

An api to interact with clients/connections. All functions in this module
return a ``Promise``-instance.

``Client.send(ref, message)``
`````````````````````````````

Send a `message` to a specific client identified by `ref`.

Example::

    Client.send(event.ref, "Welcome!");


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
