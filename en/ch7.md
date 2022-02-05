# Suggested Setups and Telemetry Pipelines

Now that we understand the individual building blocks that make up OpenTelemetry, how should we combine them into a robust pro‐ duction pipeline?

The answer depends upon where you are starting from. OpenTelemetry is modular and designed to work at a variety of dif‐ ferent scales. You need to use only the pieces that feel relevant. That said, we’ve created a suggested roadmap for you to follow.

Installing the OpenTelemetry Client

It is possible to use the OpenTelemetry client on its own without deploying a Collector. This basic setup is usually a sufficient starting point for greenfield deployments, both for testing and for initial production. The OpenTelemetry SDK can be configured to transmit telemetry directly to most observability services.

Picking an Exporter

By default, OpenTelemetry exports data using OTLP. The SDK ships with exporters for several common formats: Zipkin, Prometheus, StatsD, etc. If the observability backend you are using does not sup‐ port OTLP natively, one of these other formats will most likely be supported. Install the correct exporter and send data directly to your backend system.

Installing Library Instrumentation

In addition to the SDK, OpenTelemetry instrumentation must be installed for all HTTP clients, web frameworks, databases, and message queues contained within the application. If one of these libraries is missing instrumentation, context propagation will break, leading to incomplete traces and confusing data.

In some languages, such as Java, instrumentation can be installed automatically, which makes this easier. Make sure to learn how OpenTelemetry manages instrumentation in the programming lan‐ guage you are using and double-check that instrumentation is correctly installed in your application.

Choosing a Propagator

It is also important to double-check which propagators your system needs. By default, OpenTelemetry uses the W3C trace-context and baggage propagators. However, if your application needs to commu‐ nicate with services that use a different tracing propagator, such as Zipkin’s B3 or AWS’s X-Amzn, then change the OTEL_PROPAGA‐ TORS configuration to include this additional propagator.

If OpenTelemetry is eventually going to replace these other tracing systems, I recommend running with both trace-context and the additional tracing propagator. This will allow for a seamless transi‐ tion to W3C standards as you progressively replace the older systems in your deployment.

Deploying a Local Collector

While it is possible for some systems to make do with just the client, your operational experience can be improved by adding a local Col‐ lector to the machine your application is running on.

Running a local Collector has a number of benefits, as shown in

Figure 7-1. The Collector can generate machine metrics (CPU, RAM, etc.), a critical piece of telemetry. The Collector can also per‐ form any needed data-processing tasks, such as scrubbing PII from tracing and log data.

*Figure 7-1. A local Collector (C) receives telemetry from local applications (A) while collecting host metrics (H).* *The* *Collector exports the combined telemetry to the next stage in the pipeline.*

Running a Collector allows most telemetry configuration to be moved out of your application. Telemetry configuration is often deployment specific, not application specific. The SDK can simply be set to use a default configuration that always exports OTLP data to a predefined local port. By managing the local Collector, the oper‐ ator can make configuration changes without needing to coordinate with application developers or reboot the application. This is espe‐ cially helpful when moving applications through a complicated CI/CD (continuous integration / continuous delivery) pipeline, where telemetry needs to be handled differently in various staging and load-testing environments.

Quickly sending telemetry data to a local Collector can work as a buffer to handle back-pressure and ensure that the buffered teleme‐ try data will not be lost when the application crashes.

Deploying a Collector-Processing Pool

If your local Collector begins to perform a significant amount of buffering and data processing, it can steal resources from your application. This can be solved by deploying a pool of machines that only run Collectors, which sit behind a load balancer, as shown in Figure 7-2. The size of the collector pool can now be managed based on data throughput.

*Figure 7-2. Applications (A) can send telemetry to a pool of Collectors (C) by using a router or load balancer.*

The local Collectors can now have their processors switched off to free up resources. They continue to collect machine-level telemetry to act as a forwarding mechanism for OTLP coming from local applications.

Adding Additional Processing Pools

Sometimes a single Collector pool isn’t enough. Some tasks may need to scale at different rates. Splitting the Collector pool into a pipeline of more specialized pools may allow for more efficient and manageable scaling strategies, as the workload in each specialized Collector pool becomes more predictable.

Once you’ve reached this scale, there are no more deployment rec‐ ipes. The specialized needs of large-scale systems are often unique, and these requirements will drive the topology of your observability pipeline. Use the flexibility provided by the Collector to tailor every‐ thing to your needs. I recommend benchmarking the resource consumption of each Collector configuration and using this infor‐ mation to create elastic, auto-scaling Collector pools.

Managing Existing Telemetry with the Collector

The roadmap described above works for rolling out OpenTelemetry. But how should you handle existing telemetry? Most running systems already have some form of metrics, logs, and (possibly) tracing. And large, long-lived systems often end up with a patch‐ work of multiple telemetry solutions. Different components may have been built in different eras, and some components may have been inherited from an outside source, such as an acquisition. This can result in a messy situation, from an observability point of view.

Even in complex legacy situations such as this, it is still possible to transition to OpenTelemetry without downtime or having to rewrite every service all at once. The secret is to first deploy a Collector that acts as a transparent proxy.

In the Collector, set up receivers for receiving every type of teleme‐ try your system currently produces, connected to exporters that send telemetry in the exact same formats. A StatsD receiver is con‐ nected to a StatsD exporter, a Zipkin receiver is connected to a Zipkin exporter, and so on. This transparent proxy can then be rolled out incrementally, without creating disruption. Once all tele‐ metry is being mediated by these Collectors, additional processing can be introduced. Even before you switch your instrumentation over to OpenTelemetry, you may find that these Collectors are a helpful way to manage and organize your current patchwork teleme‐ try system. Figure 7-3 shows a collector processing data from a variety of sources.

*Figure 7-3.* *The* *Collector can help manage complex, patchwork observability systems, which send data in a variety of formats to a variety of storage systems.*

To begin switching services over to OpenTelemetry, an OTLP receiver can be added to the Collector, connected to the existing exporters. As services switch to using the OpenTelemetry clients, they send OTLP to the Collectors, which will translate OTLP back into the same data that these systems were producing previously. This allows OpenTelemetry to be rolled out incrementally by sepa‐ rate application teams, without disruption.

Switching Providers

Once all telemetry traffic is being sent through Collectors, switching to a new observability backend becomes easy: simply add an exporter to the Collectors that sends data to the new system you want to try and tee the telemetry off to both the old system and the new system. By sending data to both systems, you create an overlap in coverage. If you like the new system, you can then decommission the old system after some time, to avoid creating a gap in visibility. Figure 7-4 illustrates this process.

*Figure 7-4. Using a Collector to migrate between observability backends without a break in service.*

It is also possible to use OpenTelemetry to run a “bake-off” between multiple providers. You can tee the telemetry off to multiple systems at the same time and compare systems directly to see which one best suits your needs.