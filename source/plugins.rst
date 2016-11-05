Plugins
*******

Plugins are custom modules which are given access to the configuration (from ``config``) and the core event emmitter. Typically used for custom reporting (such as `StatsD <https://github.com/shoreditch-ops/artillery-plugin-statsd>`_) but feel free to get creative!

Configuration
-------------

Plugins are configured in the configuration section
::

  {
    "config": {
      // ...
      "plugins": {
        "statsd": {
          "host": "localhost",
          "port": 8125,
          "prefix": "artillery"
        }
      }
    }
    // ...
  }

The code above would load the plugin named ``artillery-plugin-statsd``. Plugins are expected to follow the naming convention of ``artillery-plugin-somename``. The plugin configuration values (such as ``host``, ``port``, and ``prefix`` above) are specific to each plugin. See the individual plugin's documentation for more details.

.. note:: Plugins must be installed seperatly. If artillery is installed globally, install your plugins globally. If artillery is installed somewhere else, install your plugin in that project.

Build your own
--------------

Plugins are expected to be standard CommonJS modules and are loaded via the ``require()`` method. As mentioned above you will get the script configuration and core event emmitter. Here is what a plugin might look like:
::

  //
  // my-cool-plugin.js
  //
  module.exports = MyCoolPlugin

  function MyCoolPlugin(config, ee){

    var host = config.plugins.myCoolPlugin.host;

    ee.on("stats", function(stats){
      // push intermediate stats to configured host
    });

    ee.on("done", function(stats){
      // push aggregate stats to configured host
    });

    ee.on("phaseStarted", function(spec){
      // Do a flip because you are really happy
    });

    ee.on("phaseCompleted", function(spec){
      // Play a really annoying noise to let everyone know your code is working
    });
  }

The event emmitter is controlled by `artillery-core <https://github.com/shoreditch-ops/artillery-core>`_. Some of the events you can expect to see are:

- ``stats`` - Triggered every second during the test. Passes a stats object for the previous second.
- ``done`` - Triggered on test completion. Passes a stats object for the whole test.
- ``phaseStarted`` - Triggered when a phase is started. Passes the phase definition from the script.
- ``phaseCompleted`` - Triggered when a phase is completed. Passes the phase definition from the script.

The stats objects mentioned above come from the `definition in artillery-core <https://github.com/shoreditch-ops/artillery-core/blob/master/lib/stats2.js>`_.

.. note:: As is, the stats object may not appear useful, but calling the ``result()`` function will give you a new object with the stats you are used to seeing in the output.

Known Plugins
-------------

Here are some known plugins that people have built for artillery. Go check them out!

`artillery-plugin-statsd <https://github.com/shoreditch-ops/artillery-plugin-statsd>`_ - Publish the stats produced by Artillery CLI to StatsD in real-time

`artillery-plugin-influxdb <https://github.com/shoreditch-ops/artillery-plugin-influxdb>`_ - Records response data into InfluxDB

`artillery-plugin-cloudwatch <https://github.com/shoreditch-ops/artillery-plugin-cloudwatch>`_ - Records response data into cloudwatch

`artillery-plugin-aws-sigv4 <https://github.com/shoreditch-ops/artillery-plugin-aws-sigv4>`_ - Signs HTTP requests using the AWS Signature V4 specification.

`artillery-plugin-datadog <https://github.com/bigbank-as/artillery-plugin-datadog>`_ - Reports load test results to Datadog
