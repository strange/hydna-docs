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
        "environment": {
            "password": "secret",
            "debug-mode": false,
            "message-prefix": "data >>> "
        },
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
        "environment": {
            "password": "secret",
            "debug-mode": false,
            "message-prefix": "data >>> "
        },
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

``Domain``
~~~~~~~~~~~

An api to interact with the domain.


``Domain.hostname()``
`````````````````

Returns the name of domain. E.g. ``"test.hydna.net"``. This is equal to call
``Domain.env("net.hostname")``.

Example::

    function onmessage(event) {
        Channel.send(event.path, `${Domain.hostname()}: ${event.data}`);
    }


``Domain.env([name])``
``````````````````````````

Returns the value of the domain environmental variable ``name`` if provided, or
all key/values if omitted.

Predefined environmental variables

======================== ======================================================
Name                     Description
======================== ======================================================
``net.hostname``         The hostname of the domain (string)
``net.tls.enabled``      Indicates if domain supports TLS or not (bool)
``net.limit.sockets``    Max connections of domain (number)
``net.limit.message``    The largest size of a message in bytes (number)
``net.limit.requests``   The maximum number of simultaneously incomming
                         HTTP-requests (number)
``net.limit.response``   The largest size of an outgoing http-response in
                         bytes (number)
``script.timeout``       Number of milliseconds a script can run without
                         throwing a timeout error (number)
``cache.enabled``        Indicates if the Cache-module is enabled or not (bool)
``cache.limit.size``     The maximum capacity (in bytes) of the domain
                         cache (number)
``cache.limit.key``      Cache key size limit in bytes (number)
``cache.limit.value``    Cache value size limit in bytes (number)
``http.enabled``         Indicates if the Http-module is enabled or not (bool)
``http.limit.reqeusts``  The maximum number of outgoing HTTP-requests at
                         any given point(number)
``timezone.offset``      Returns the timezone offset of domain (number)
======================== ======================================================

Additional variables can be defined by user in ``mainfest.json``.

Example::

    const maxconns = Domain.env("net.limit.sockets");
    console.log(`Domain support ${maxconns} of simultaneously connections");

Example (list all environmental variables)::

    const env = Domain.env();
    console.log("Domain variables:");
    console.log(env);

Example (user-defined variables)::

    function onmessage(event) {
        const prefix = Domain.env("message-prefix");
        Channel.send(event.path, prefix + event.data);
    }

Example (using user-defined variable "debug-mode")::

    function onmessage(event) {
        if (Domain.env("debug-mode")) {
            console.log("Incomming message :: " + event.data);
        }
    }

Example (using user-defined variable "password")::

    function onopen(event) {
        if (event.querystring === Domain.config("password")) {
            event.allow();
        } else {
            event.deny();
        }
    }


``Domain.restart([reason])``
````````````````````````````

Restarts the domain by gently disconnects all connections. The optional
``reason`` is sent as reason to the client.

The ``Cache`` is reset once started again.

Example::

    function onmessage(event) {
        if (event.data === "restart") {
            Domain.restart("Restarting domain...");
        }
    }


``Domain.restartAfter(timeout, [reason])``
````````````````````````````

Restarts the domain by gently disconnects all connections after ms
``timeout``. The optional ``reason`` is sent as reason to the client.

The ``Cache`` is reset once started again.

Please note that this command cannot be canceled.

Example::

    function onmessage(event) {
        if (event.data === "restart") {
            Channel.send("/", "The domaini will shutdown in 3 sec...");
            Domain.restartAfter(3000, "Restarting domain...");
        }
    }


``Domain.kill()``
````````````````````````````

Restarts the domain without waiting connections to be disconnected or to wait
for running scripts to finish.

Note: This command should be concidered to be used as a last resort.

Example::

    function onmessage(event) {
        if (event.data === "kill") {
            console.log("something is terriable wrong, killing domain");
            Domain.kill();
        }
    }



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


``Cache``
~~~~~~~~~

An API to interact with a domain's key-value cache. All functions in this
module return a ``Promise``-instance.


``Cache.get(key)``
``````````````````

Get value associated with ``key``. Will return an error if the value is 
a hash, a list, or if there is no value associated with the key.

Example::

    function onopen(event) {
        Cache.get("welcome-message")
            .then((value) => Channel.sendAfter(event.ref, 1, value));
    }


``Cache.set(key, value)``
`````````````````````````

Destructively set value of ``key`` to ``value``. Any existing value --
regardless of type -- will be overwritten.

Example::

    Cache.set("secret-key", "password")
        .then(() => Console.log("Set was successfull"))
        .catch((error) => Console.log("An error occured %s", error.message));


``Cache.del(key)``
``````````````````

Permanently delete value at ``key``, regardless of type.

Example::

    Cache.del("key").then(() => Console.log("Key is now deleted"));


``Cache.incr(key, [max])``
``````````````````````````

Increase value associated with ``key`` by ``1``. The initial value is set to
``0`` if the key does not exist. Will return an error if there is an existing
value at ``key`` that cannot be converted into an integer. A successfull
invocation will return the new value.

Example::

    const MAX_CONNECTIONS = 10;
    function onopen(event) {
        Cache.incr("path-counter", MAX_CONNECTIONS)
            .then(() => event.allow())
            .catch(() => event.deny())
    }


``Cache.decr(key, [min])``
``````````````````````````

Works like ``incr()`` above, but decreases the value of ``key`` by ``1``.

Example::

    function onclose(event) {
        Cache.decr("path-counter");
    }


``Cache.push(key, value)``
``````````````````````````

Adds ``value`` to the end of list at ``key``. Will return an error if the key
exists and the value is not a list. A successfull invocation returns the
current length of the list.

Example::

    function onmessage(event) {
        if (event.data.startsWith("add-job")) {
            const jobname = event.data.substr(6);
            Cache.push("work-queue", jobname);
        }
    }



``Cache.pop(key)``
``````````````````

Pop a value from the end of list at ``key``. If the popped item was the last
item, the key is deleted. An error will be returned if  an attempt is made to
pop an item from a value that is not a list.

Example::

    function onevent(event) {
        if (event.handler === "workqueue-tick") {
            Cache.pop("work-queue").then((value) => {
                // Do something with job
            });
        }
    }


``Cache.unshift(key, value)``
`````````````````````````````

Add ``value`` to the begining of list at ``key``.



``Cache.shift(key)``
````````````````````

Remove and return a value from the beginning of list at ``key``. If the item
was the last item, the key is deleted. An error will be returned if  an
attempt is made to pop an item from a value that is not a list.

``Cache.popunshift(fromKey, toKey)``
````````````````````

Pop value from ``fromKey`` (which must be a non-empty list) and unshift it to
``toKey`` (which must be unsert or a list) as an atomic operation.

``Cache.range(key, start, [length])`` (TBA)
```````````````````````````````````````````

Return a range of elements in list at ``key`` starting at ``start`` (inclusive,
zero-based). If start is a negative number the range will start that many
elements from the end of the list.


``Cache.trim(key, start, [length])`` (TBA)
``````````````````````````````````````````

Works basically the same as range() (see above), but trims the list to contain
only elements in the specified range (i.e. it removes all elements not in the
range from the list).


``Cache.hget(key, field)``
``````````````````````````

Get value from ``field`` of hash ``key``.


``Cache.hset(key, field, value)``
`````````````````````````````````

Set ``field`` of hash ``key`` to ``value``. Will return ``true`` if the
``field`` was not present in the hash prior to the invocation.


Example::

    function onopen(event) {
        const username = event.querystring;
        Cache.hset("connected-users", event.ref, username)
            .then(() => event.allow())
            .catch(() => event.deny());
    }


``Cache.hkeys(key)``
````````````````````

Return a list of all fields in hash associated with ``key``.

Example::

    function onmessage(event) {
        if (event.data === "list-users") {
            Cache.hkeys("connected-users")
                .then((keys) => Socket.send(event.ref, JSON.stringify(keys)));
        }
    }


``Cache.hvalues(key)``
``````````````````````

Return a list of all values in hash associated with ``key``.


``Cache.hdel(key, field)``
``````````````````````````

Delete ``field`` in hash associated with ``key``.

Example::

    function onclose(event) {
        Cache.hdel("connected-users", event.ref);
    }
