Script Reference
****************

Artillery runs scripts written in JSON or YAML. Scripts configure the target, default behavior, and scenarios for virtual users that Artillery will generate.

An Artillery script has two sections, ``config`` and ``scenarios``.

``config``
##########

- ``target`` - base URL for all requests in this script. Examples: ``http://myapp.staging.local`` or ``ws://127.0.0.1``.
- ``environments`` - specify a list of environments, and associated target URLs; see `environments <script_reference.html#environments>`_
- ``phases`` - specify the duration of the test and frequency of requests; see `phases <script_reference.html#phases>`_
- ``payload`` - used for importing variables from a CSV file; see `payloads <script_reference.html#payloads>`_
- ``plugins`` - a list of plugins to load; see `Plugins <plugins_links.html>`_
- ``defaults`` - set default headers that will apply to all HTTP requests
- ``ensure`` - a list of latency metrics and their desired value(s). Causes Artillery to return with a non-zero error code if not met. Useful for CI
- ``tls`` - configure how Artillery handles self-signed certificates. See `HTTP Reference <testing_http.html>`_
- ``timeout`` - number of seconds to wait for the server to start responding (send response headers and start the response body). Default value is ``10``.


Environments
============

You can specify a number of named environments in the configuration. Example:
::

    {
      "config": {
          "target": "http://wontresolve.local:3003",
          "phases": [
            {"duration": 10, "arrivalRate": 1}
          ],
          "environments": {
            "production": {
              "target": "http://wontresolve.prod:44321"
            },
            "staging": {
              "target": "http://127.0.0.1:3003",
              "phases": [
                {"duration": 20, "arrivalRate": 1}
              ]
            }
          }
      },
      "scenarios": [
         ...

Choose an environment on the command line with the ``-e`` argument; Example ``-e staging``.

Phases
======

A phase is a period of the load test that can be named and have timing parameters. This can be used to do things like having a ramp-up phase following by a more intense phase.

``config.phases`` is an array of phase objects that Artillery goes through sequentially. Phases typically come in three flavors:

- ``arrivalRate`` - specify the arrival rate of virtual users for a duration of time.
  - A linear "ramp" in arrival can be also be created with the ``rampTo`` option
- ``arrivalCount`` - specify the number of users to create over a period of time.
- ``pause`` - pause and do nothing for a duration of time.

Examples
--------

Create 50 virtual users every second (on average) for 5 minutes
::

    { "duration": 300, "arrivalRate": 50 }

Ramp up arrival rate from 10 to 50 users/second over 2 minutes
::

    { "duration": 120, "arrivalRate": 10, "rampTo": 50 }


Create 20 virtual users in 60 seconds (one every 3 seconds on average)
::

    { "duration": 60, "arrivalCount": 20 }


Pause and do nothing for 10 seconds
::

    { "pause": 10 }


Arrival phases can be named, for example
::

    { "duration": 180, "arrivalRate": 10, "name": "Warm-up" }


Putting it all together
::

    {
      "config": {
        "target": "http://myapp.staging.local",
        "phases": [
          {"duration": 300, "arrivalRate": 5, "name": "Warm-up"},
          {"pause": 10},
          {"duration": 60, "arrivalCount": 30 },
          {"duration": 600, "arrivalRate": 50, "name": "High load phase"}
        ]
      },
      "scenarios": [
        // scenario definitions
      ]
    }

Payloads
========

In some cases it is useful to be able to inject data from external files into your test scenarios. For example, you might have a list of usernames and passwords that you want to use to test the authorization endpoint of your API.

Payload files are in the CSV format and Artillery allows you to map each of the rows to a variable name that can be used in scenario definitions. For example:
::

    {
      "config": {
        // other config...
        "payload": {
          "path": "users.csv", // path is relative to the location of the test script
          "fields": ["username", "password"]
        }
      },
      "scenarios": [
        {
          "post": {
          "url": "/auth",
          "json": {
            "username": "{{ username }}",
            "password": "{{ password }}"
          }
         }
        }
      ]
    }


If you have multiple CSV files, ``"payload"`` can also be an an array:

::

    "payload": [
      {
        "path": "./pets.csv",
        "fields": ["species", "name"]
      },
      {
        "path": "./urls.csv",
        "fields": ["url"]
      }
    ]

Ordering
--------

Rows from the CSV file are picked *at random* by default. To iterate through the rows in sequence (looping around and starting from the beginning after the last row has been reached), set the ``"order"`` attribute to ``"sequence"``:
::

    {
      "config": {
        // other config...
        "payload": {
          "path": "users.csv", // path is relative to the location of the test script
          "fields": ["username", "password"],
          "order": "sequence"
        }
      },
      "scenarios": [
        // rest of the script

Ensure
======

The ``ensure`` property can be used to validate certain latencies stayed under a desired value
::

  {
    "config": {
      // other config...
      "ensure": {
        "p95": 3000,
        "max": 5000
      }
    },
    "scenarios": [
      // rest of the script

With the above configuration, Artillery will check that ``p95`` is less than 3000ms and ``max`` is less than 5000ms on test completion. If either case fails, artillery will exit with a non-zero exit code.

.. note:: While ``p95`` and ``max`` were used in the example, any of the reported latency values can be used.


``scenarios``
#############

The ``scenarios`` key is an array that must exist in the root of the script. It contains a list of ``flow`` objects.

A scenario is a sequence of steps that need to be run sequentially, and represents a sequence of calls generated by a simulated user.

``name``
========

You can give your scenario a descriptive name with this attribute, e.g. ``"search for a product and get its details"``

``weight``
==========

Weights allow you to specify that some scenarios should be picked more often than others. If you have three scenarios with weights ``1``, ``2``, and ``5``, the scenario with the weight of ``2`` is twice as likely to be picked as the one with weight ``1``, and 2.5 times less likely than the one with weight ``5``. Or in terms of probabilities:

- scenario 1: 1/8 = 12.5% probability of being picked
- scenario 2: 2/8 = 25% probability
- scenario 3: 5/8 = 62.5% probability

Weights are optional, and if not specified are set to ``1`` (so each scenario is equally likely to be picked).

``flow``
========

A "flow" is an array of operations that a virtual user performs. There are different options depending on what type of endpoint you are testing. Please see the sections on `Testing HTTP <testing_http.html>`_, `Testing Socket.io <testing_socketio.html>`_, and `Testing websockets <testing_websockets.html>`_ for more information more pertanent to your type of testing. You can also check out the `scripts used for testing <https://github.com/shoreditch-ops/artillery-core/tree/master/test/scripts>`_ in artillery-core.

``think``
---------

You can use a ``think`` step in a flow to pause the execution of the scenario for N seconds, e.g.:
::

    { "think": 1 }

will pause for 1 second before continuing with the next request.
