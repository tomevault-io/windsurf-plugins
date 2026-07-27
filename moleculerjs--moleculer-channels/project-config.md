---
trigger: always_on
description: This is a codebase for message delivery for MoleculerJS services via external queue/channel/topic. Unlike moleculer built-in events, this is **not** a fire-and-forget solution. It's a persistent, durable and reliable message sending solution. The module uses an external message queue/streaming server that stores messages until they are successfully processed.
---

# Context

This is a codebase for message delivery for MoleculerJS services via external queue/channel/topic. Unlike moleculer built-in events, this is **not** a fire-and-forget solution. It's a persistent, durable and reliable message sending solution. The module uses an external message queue/streaming server that stores messages until they are successfully processed.
The module supports multiple queue/streaming servers, including Redis, RabbitMQ, NATS JetStream, Kafka. It provides a unified interface for sending and receiving messages across different brokers.

# Directory Structure

```
.
├── index.js
├── package.json
├── examples
├── src
│   ├── adapters
│   │   ├── amqp.js
│   │   ├── base.js
│   │   ├── fake.js
│   │   ├── index.js
│   │   ├── kafka.js
│   │   ├── nats.js
│   │   └── redis.js
│   ├── constants.js
│   ├── index.js
│   └── tracing.js
├── test
│   ├── docker-compose.yml
│   ├── integration
│   │   └── index.spec.js
│   └── unit
│       └── index.spec.js
```

# Key Files

-   `index.js`: Entry point of the module.
-   `src/adapters/`: Contains adapter implementations for different message brokers (Redis, RabbitMQ, NATS, Kafka). All of adapters extend from `base.js`.
-   `src/constants.js`: Defines constants used across the module such as metrics names, Redis header keys, etc.
-   `src/tracing.js`: Implements tracing middleware for tracing the message flow. The middleware is implemented via MoleculerJS middleware, and it works with any adapter. It is initialized via `created()` [lifecycle method](https://moleculer.services/docs/0.15/middlewares#created-broker-async) of the service broker.
-   `src/index.js`: Main file that implements `created()`, `serviceCreated()`, `serviceStopping()`, and `stopped()`. It is responsible for creating broker-level methods like `sendToChannel()` (configurable via `sendMethodName` option), exposing raw adapter via `channelAdapter` (configurable via `adapterPropertyName` option), registering metrics, registering service topics listeners and cleaning up resources on broker stop.
-   `test/`: Contains integration tests. Integration tests use Docker Compose to spin up required message brokers. The focus of the tests is to verify the integration with different message brokers and ensure that messages are sent and received correctly and that from Moleculer's perspective, the behavior is consistent across different adapters.
-   `package.json`: Defines the module's dependencies, scripts, jest configuration, and metadata
-   `examples/`: Contains different ways of using the module with different adapters and configurations.

The core file from which the adapters are created from is `src/adapters/base.js`. All adapters extend from this base class and implement the required methods for sending and receiving messages.

** Skeleton of `src/adapters/base.js` **

```js
class BaseAdapter {
    /**
     * Constructor of adapter
     * @param  {Object?} opts
     */
    constructor(opts) {
        /** @type {BaseDefaultOptions} */
        this.opts = _.defaultsDeep({}, opts, {
            consumerName: null,
            prefix: null,
            serializer: "JSON",
            maxRetries: 3,
            maxInFlight: 1,
            deadLettering: {
                enabled: false,
                queueName: "FAILED_MESSAGES"
            }
        });

        /**
         * Tracks the messages that are still being processed by different clients
         * @type {Map<string, Array<string|number>>}
         */
        this.activeMessages = new Map();

        /** @type {Boolean} Flag indicating the adapter's connection status */
        this.connected = false;
    }

    /**
     * Initialize the adapter.
     *
     * @param {ServiceBroker} broker
     * @param {Logger} logger
     */
    init(broker, logger) {
        this.broker = broker;
        this.logger = logger;
        this.Promise = broker.Promise;

        if (!this.opts.consumerName) this.opts.consumerName = this.broker.nodeID;
        if (this.opts.prefix == null) this.opts.prefix = broker.namespace;

        this.logger.info("Channel consumer name:", this.opts.consumerName);
        this.logger.info("Channel prefix:", this.opts.prefix);

        // create an instance of serializer (default to JSON)
        /** @type {Serializer} */
        this.serializer = Serializers.resolve(this.opts.serializer);
        this.serializer.init(this.broker);
        this.logger.info("Channel serializer:", this.broker.getConstructorName(this.serializer));

        this.registerAdapterMetrics(broker);
    }

    /**
     * Register adapter related metrics
     * @param {ServiceBroker} broker
     */
    registerAdapterMetrics(broker) {
        if (!broker.isMetricsEnabled()) return;

        broker.metrics.register({
            type: METRIC.TYPE_COUNTER,
            name: C.METRIC_CHANNELS_MESSAGES_ERRORS_TOTAL,
            labelNames: ["channel", "group"],
            rate: true,
            unit: "msg"
        });


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moleculerjs/moleculer-channels](https://github.com/moleculerjs/moleculer-channels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
