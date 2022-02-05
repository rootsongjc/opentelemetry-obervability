# OpenTelemetry: An Architectural Overview

Chapter 2 describes the data model needed to enable automated analysis, and Chapter 4 describes additional requirements to sup‐ port native OSS instrumentation and empower autonomy among roles (application owner, operator, and responder). This is our con‐ ceptual model for modern observability.

In the remainder of this report, we describe a real-world implemen‐ tation of this new model, OpenTelemetry. This chapter describes all of the individual components that constitute the OpenTelemetry telemetry pipeline. Subsequent chapters will describe the stability guarantees, suggested setups, and deployment strategies for real- world adoption of OpenTelemetry. Further details about the project can be found in the appendixes.

## Signals

OpenTelemetry specification is organized into distinct types of tele‐ metry, which we call *signals*. The primary signal is tracing. Logs and metrics are other examples. Signals are the most fundamental unit of design in OpenTelemetry.

Each additional signal is first developed independently and then integrated with tracing and other relevant signals. This separation allows new, experimental signals to be developed without affecting the compatibility guarantees of signals that have already become stable.

OpenTelemetry is a *cross-cutting concern* that follows the execution of a transaction as it passes through each library and service. To accomplish this, all signals are built on top of a lower-level context propagation system, which provides a place for signals to store any transaction-level data they need to associate with the code that is currently executing. Because the context propagation system is cleanly separated from tracing, other cross-cutting concerns may also make use of it. Figure 5-1 illustrates this layered architecture.

*Figure 5-1. All OpenTelemetry signals are built on top of a shared context propagation system. Other, non-observability cross-cutting concerns may also use the context propagation mechanism to transport their data through a distributed system.*

Context

The context object is a key-value store associated with an execution context, such as a thread or coroutine. How this is implemented is language dependent, but OpenTelemetry provides a context object in every language.

Signals store their data in the context object. Because OpenTeleme‐ try API calls always have access to the entire context object, it is pos‐ sible for signals to become integrated and share data under the hood, without requiring API changes. For example, if both the trac‐ ing and the metrics signals are enabled, recording a metric may automatically create a trace exemplar. Likewise with logging: logs will automatically be bound to the current trace, if there is one.

Propagators

In order for distributed tracing to work, the trace context must be shared by every service that participates in the transaction. Propaga‐ tors accomplish this by serializing and deserializing the context object, allowing the signals to follow their transactions across net‐ work requests.

Tracing

The OpenTelemetry tracing system is based on OpenTracing and OpenCensus. Both of these systems, along with the popular Zipkin and Jaeger projects, are based on the Dapper tracing system devel‐ oped at Google. OpenTelemetry seeks to be compatible with all of these Dapper-based systems.

OpenTelemetry tracing includes a concept called *links*, which allow separate traces to be combined into a larger graph. This is used to connect transactions to background processing and to observe large asynchronous systems such as Kafka and AMQP.

Metrics

Metrics is a big space, containing a wide variety of approaches and implementations. The OpenTelemetry metrics signal is designed to be fully compatible with Prometheus and StatsD.

Metrics include trace exemplars, automatically associating metrics with a sampling of the traces that generated them. Associating met‐ rics and traces by hand is often a tedious and error-prone task, and performing this task automatically will save operators a significant amount of time.

Logs

OpenTelemetry combines a highly structured logging API with a high-speed log-processing system. Existing logging APIs can be connected to OpenTelemetry, preventing the need to re-instrument an application.

Whenever it is present, logs are automatically attached to the current trace. This makes transaction logs easy to find and allows automated analysis to find accurate correlations between logs in the same trace.

Baggage

OpenTelemetry baggage is a simple but versatile key-value system. Once data is added as baggage, it becomes accessible to all down‐ stream services. This allows useful information, such as account and project IDs, to become available later in the transaction without needing to refetch them from a database. For example, a frontend service using project ID as an index could add it as baggage, allow‐ ing backend services to also index their spans and metrics by project ID.

A good way to think about baggage is as a form of *distributed con‐ text*. Items put directly into the context object are only accessible within the current service. Much like trace context, items added as baggage are injected into network requests as headers, allowing them to be extracted by the downstream service.

Like the context object, baggage is not an observability tool per se. It is more of a general purpose data storage and transmission system. Beyond observability, other cross-cutting tools—feature flagging, A/B testing, and authentication, for example—could use baggage to store any state they need to track the current transaction.

However, baggage comes at a cost. Because every item added must be encoded as a header, every item you add will increase the size of every subsequent network request in the transaction. That’s why we call it baggage. I recommend that baggage be used sparingly, as part of a cross-cutting concern. Baggage should not be used as a “conve‐ nient” alternative to well-defined service APIs for explicitly sending parameters to downstream applications.

## The OpenTelemetry Client Architecture

An application is instrumented with OpenTelemetry by installing a collection of the software libraries: the API, the SDK (software development kit), SDK plug-ins, and library instrumentation. This set of libraries is referred to as the OpenTelemetry client. Figure 5-2 shows the relationships among these components.

*Figure 5-2. OpenTelemetry client architecture. To help manage dependencies, OpenTelemetry separates the implementation from the API used by instrumentation.*

In many languages, OpenTelemetry provides installers, which help to automate the installation and setup of the OpenTelemetry clients. However, the amount of automation available is language depen‐ dent. In Java, OpenTelemetry provides a Java agent, which com‐ pletely automates setup by dynamically injecting all the necessary components. In Go, OpenTelemetry packages must be installed and initialized by writing code, like any other Go package. Python, Ruby, and NodeJS are in between, providing varying degrees of automation.

When learning OpenTelemetry, it is important to understand how setup works in the language you are using. In particular, be sure to learn how instrumentation should be installed, as this varies consid‐ erably from language to language.

Check out the client documentation for more details on getting started.

## Client Architecture: The Instrumentation API

The *OpenTelemetry API* refers to the set of components used to write instrumentation. The API is designed to be embedded directly into OSS libraries as well as applications. This is the only part of OpenTelemetry that shared libraries and application logic should take a direct dependency on.

Providers

The API is completely separate from any implementation. When an application starts, an implementation can be loaded by registering a provider for each signal. The providers become the receivers of all API calls.

When no providers are loaded, the API defaults to a no-op provider. This makes OpenTelemetry instrumentation safe to include in shared libraries. If the application does not use OpenTelemetry, the API calls simply become no-ops and do not incur any overhead.

For production use, we recommend using the official OpenTeleme‐ try providers, which we refer to as the *OpenTelemetry SDK*.

### Why Have Multiple Implementations?

The separation of API and implementation has a number of bene‐ fits. But would that have any real meaning if users were forced to always install the official OpenTelemetry SDK? Is there ever a rea‐ son to install another implementation? The SDK is already very extensible.

We believe there is. While we want OpenTelemetry instrumentation to be universal, we do not believe it is possible to build a single implementation that will be ideal for all use cases. As good as we believe the OpenTelemetry SDK is, there should always be an option to use another implementation. Flexibility in implementa‐ tion is a key feature to providing a universal instrumentation API.

For one thing, this separation ensures that OpenTelemetry will never create an insurmountable dependency conflict. There is always the option to load an implementation that includes a differ‐ ent dependency chain.

Another reason is performance. The OpenTelemetry SDK is an extensible, general-purpose framework. While the SDK is designed to be as performant as possible, extensibility and performance always come as a trade-off. For example, creating a binding to the OpenTelemetry C++ SDK via foreign function interface has the potential to be a very efficient option for dynamic scripting lan‐ guages such as Ruby, Python, and Node.js.

There are also streaming architectures that show promising perfor‐ mance boosts. With many of these optimized solutions, the ability to write plug-ins and lifecycle hooks would be severely limited; the data structures needed to support those types of features would be nonexistent in these optimized solutions. Ultimately, there is no “perfect implementation”; there are only trade-offs.

The API/SDK separation is a key design choice, which the project makes heavy use of. For example, besides the SDK, every language has a no-op implementation, which is installed by default. There also is a Fake/Mock implementation available, which we use for testing. And there is the potential for even more creative implementations—for example, building developer tools for distributed systems, such as a live debugger, which works across network boundaries.

Client Architecture: The SDK

The OpenTelemetry project provides an official implementation for the OpenTelemetry API, which we call the OpenTelemetry SDK. The SDK implements the OpenTelemetry API by providing a plug-in framework. The tracing SDK is described below; a similar architec‐ ture is applied to metrics and logs.

The basic data structure is a lock-free SpanData object. The Span‐ Data object is created when a user starts a span, and it is built up automatically as the user adds attributes and events. Once a span is ended, the SpanData object will no longer be updated and can be safely passed to a background thread.

The SDK plug-in architecture is organized as a pipeline. For tracing, that pipeline consists of a chain of SpanProcessors. Each processor gets synchronous access to the SpanData object twice: once when the span starts and again after it has ended. Samplers, log appenders, and data scrubbers are examples of SpanProcessors. The last pro‐ cessor in the chain is often a BatchSpanProcessor, which manages a buffer of completed spans. An exporter can be connected to the BatchSpanProcessor to flush batches of spans over the network to the next service in the telemetry pipeline, often sending them to either a Collector or directly to the tracing backend. Once the spans are exported, the pipeline is complete and the SpanData objects are released.

Samplers

OpenTelemetry provides several common types of sampling algo‐ rithms, including both up-front and priority-based sampling. Sam‐ pling can help control costs, but it comes at a price: you will be missing data. Before enabling any kind of sampling algorithm, it is important to check what types of sampling are supported by the analysis tools you plan to use. Unexpected sampling may ruin some forms of analysis. Some tools require their own sampling plug-ins. For example, AWS X-Ray uses its own sampling algorithm, which is available as an AWS-specific sampling plug-in.

Exporters

The OpenTelemetry provides exporters for OTLP (OpenTelemetry Protocol), Jaeger, Zipkin, Prometheus, and StatsD. Additional exporters maintained by third parties can be found in the OpenTelemetry-Contrib repositories (repos) in each language. Use the OpenTelemetry registry to find out which plug-ins are currently available.

Client Architecture: Library Instrumentation

To work properly, OpenTelemetry requires end-to-end instrumenta‐ tion. This is not optional: if critical libraries do not include instru‐ mentation, context propagation will be broken.

Generally speaking, the libraries that must be instrumented include HTTP clients, HTTP servers, application frameworks, messaging/ queueing systems, and database clients. These libraries often play a role in context propagation.

• HTTP clients must create a *client span* to record the request. Clients must also use a propagator to inject the current context into the request as a set of HTTP headers.

- HTTP servers (application frameworks) must use a propagator to extract the context from the HTTP headers. The extracted context is used to create a *server span*, which is set as the cur‐ rently active span, which wraps all of the application code.

- Likewise, senders in a messaging/queuing system must use propagators to inject the context into the messages, so that trac‐ ing can continue by extracting the context on the receiver.

- Database clients must create a *database span* to record the data‐ base transaction. Once database servers are also instrumented with OpenTelemetry, database clients must also inject the con‐ text into the database request.

This requirement is one of the main reasons we would like to see OSS libraries ship with native instrumentation. In the meantime, instrumentation plug-ins are provided as *contrib packages* main‐ tained by the OpenTelemetry project or by third parties.

##  The Collector

In addition to the client described above, OpenTelemetry provides a stand-alone service called the Collector. The Collector is a flexible, configurable telemetry-processing system. The basic architecture is shown in Figure 5-3.

*Figure 5-3.* *The* *Collector has a* *configurable* *processing pipeline, which can import and export data in many common formats.*

Collector pipelines can be built to provide a number of services:

- Configuration, such as routing and data export formats. Almost all configuration options available to an OpenTelemetry client can alternatively be managed within a Collector.

- Data processing such as scrubbing, format translation, and tee‐ ing to multiple destinations.

- Buffering to help manage back pressure and network instability.

- Resource detection of the machine-level environment. Host, Kubernetes, and cloud provider details can be discovered and appended to all data received by the collector.

- Collection of host metrics such as RAM, CPU, and storage capacity.

Operators can use collectors to manage all of the deployment details relevant to the observability system without needing to interact with the application itself. Since most configuration options are deploy‐ ment specific—and are managed by the operator, not the application developer—moving telemetry configuration from the application to the Collector cleanly separates concerns.

If all routing and data-processing tasks are moved to a Collector, the OpenTelemetry SDK can be run with a much simpler configuration. By default, the SDK will send unprocessed OTLP data to a predeter‐ mined local port, where it can be received by a local Collector.

## Collector Architecture: Receivers

Collectors can be configured to receive telemetry from a variety of sources, in a variety of formats. Currently the Collector supports over forty different types of receivers! Once received, all of this data is converted into OTLP. OpenTelemetry supports both push- and pull-based receivers.

Collector Architecture: Processors

Once the receivers have converted telemetry into OTLP, a variety of processors become available. Processors can be configured to per‐ form a variety of tasks:

- Data scrubbing to remove sensitive data, such as PII (personally identifiable information).
- Data normalization, such as converting older versions of a data source into a version that matches the current dashboards and queries used by the backend.
- Routing data to specific backends based on certain attributes. For example, storing data related to EU-based users on storage systems hosted within the EU.
- Tail-based sampling, to help ensure that errors and outliers are more likely to be captured, while rate-limiting noisy and uninteresting information.

## Collector Architecture: Exporters

Once telemetry has been processed, it can be exported to a variety of backends. In the future, we expect more and more backends to sup‐ port OTLP natively. In the meantime, OTLP can be converted into many formats supported by currently popular systems. Please check the OpenTelemetry vendor page to find a list of commercial provid‐ ers that currently support OpenTelemetry.

In addition to converting telemetry into a single format, multiple exporters may be installed. Telemetry can be split apart by type and sent to separate backends—for example, sending tracing data to Jaeger and metrics data to Prometheus.

Duplicate telemetry may also be sent to multiple backends simulta‐ neously. This allows an operator to seamlessly switch from one backend to another, without any loss in service. It also allows spe‐ cialty analysis tools to receive data alongside a general purpose observability platform.

Collector Architecture: Pipelines

The Collector allows receivers, processors, and exporters to be combined into complex pipelines, which can be run concurrently. Pipe‐ lines are designed and managed via a YAML configuration file.

This configuration language is very powerful. By deploying a local Collector on every machine and connecting them to several tiers of Collector deployments configured to perform specialized processing tasks, Collectors can be used to develop a large-scale, robust telemetry system.
