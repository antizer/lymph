0.15.0
======
- Added nose2 plugin
- Fixed access to dotted config keys

0.14.0
======
- Added RPC versioning support
- Added --dump-headers option for ``lymph request``

0.13.0
======
- Added ``--json`` option for ``lymph request``
- Added a generic health check endpoint for WebServiceInterface
- Added `app_name` support for newrelic plugin

0.12.0
======
- Added monitoring for psutil metrics
- Added RPC call entries in New Relic traces

0.11.0
======
- Made rpc connection properties configurable
- Simplified collecting custom metrics

0.10.0
======
- Added retry support for event handlers.
- Added event handler tracking for the newrelic plugin.
- Fixed trace id propagation for deferred rpc calls.

0.9.0
=====
- Added support for configurable metrics tags.

0.8.0
=====
- Added lymph.task() decorator.

0.7.1
=====
- Fixed a bug that prevented custom logging configuration from being used.

0.7.0
=====
- Documentation cleanup
- Monitoring data is now published on a random port.
  It is discoverable as `monitoring_endpoint`.
- Changed datetime object serialization to always use ISO 8601 with timezone offset.
  Named timezones are no longer supported. This change is backwards incompatible.

0.6.0
=====
- Added support for RabbitMQ failover for kombu events
- Added `lymph config` command
- Added option for `lymph request` to read request body from stdin

