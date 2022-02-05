# APPENDIX B: OpenTelemetry Project Roadmap

Roadmaps go out-of-date quickly. For the most up-to-date roadmap, please see the OpenTelemetry status page. That said, here’s the state of OpenTelemetry as of this writing.

Core Components

Currently, the OpenTelemetry Tracing Signal is declared stable, and stable implementations are available in many languages.

The Metrics Signal has just been declared stable, and beta imple‐ mentations will be widely available in Q1 of 2022.

The Logging Signal is expected to be declared stable in Q1 of 2022, with beta implementations expected to be widely available in Q2 of 2022. In 2021, the Stanza project was donated to OpenTelemetry, adding highly efficient log-processing capabilities to the OpenTelemetry Collector.

The Semantic Conventions for HTTP/RPC, databases, and messag‐ ing systems are expected to be declared stable in Q1 of 2022.

The Future

Finishing the above roadmap completes the stability requirements for OpenTelemetry’s core functionality. This is a huge milestone, as it opens the door to further adoption of OpenTelemetry, including native integration with databases, managed services, and OSS libraries.Much of this integration work is already underway in the form of prototypes and beta support. With the stability of OpenTelemetry complete in the first half of 2022, I expect that there will be a surge in declarations of OpenTelemetry support in the second half of 2022, making 2022 the “year of OpenTelemetry.”

But we’re not stopping there. What’s next?

eBPF

*Extended Berkeley Packet Filter* (eBPF) is a mechanism for providing low-level network access on Windows, Linux, and other Unix-like operating systems. Leveraging eBPF would give OpenTelemetry an extremely efficient form of network monitoring that does not require any developer instrumentation.

In 2021, the Flowmill project was donated to OpenTelemetry. Flowmill is an eBPF-based observability solution designed specifi‐ cally for observing distributed systems. Flowmill developers, along with developers from the Pixie project, are working to add eBPF support to OpenTelemetry Collector. Pixie, an eBPF-based observa‐ bility tool designed specifically for Kubernetes, is a sister project to OpenTelemetry under the CNCF umbrella. Together, we are also investigating ways to further correlate low-level (layer 2) eBPF data with high-level (layer 7) distributed tracing data, which would be a first in the industry.

RUM

*Real user monitoring* (RUM) is an observability tool for describing how users interact with mobile, web, and desktop clients over long-running user sessions. RUM differs from distributed tracing due to the fact that graphical user interfaces (GUIs) tend to be reactor-based systems, which have fundamental architectural differences from transaction-based systems such as databases and web servers. Long story short: you can’t just model a user session as a trace and call it a day.

The Client Instrumentation SIG is currently developing a new RUM design, which will extend and fully integrate with OpenTelemetry’s existing distributed tracing, metrics, and logging signals.

OpenTelemetry Control Plane

Currently, OpenTelemetry Collectors and SDKs are managed as independent units, which must be restarted to change their configu‐ rations. A control plane, which reports the current status of these components and allows for live configuration changes, is currently in development by the Agent Management SIG. It would allow an operator or an automated service to dynamically control processing across the entire telemetry pipeline.

Dynamic configuration will allow for fine-grained, responsive con‐ trol over sampling, log levels, and other forms of resource manage‐ ment, thereby reducing costs. Eventually, this control plane may enable more advanced forms of tail-based sampling, sampling tech‐ niques that require coordination across the entire deployment.

Columnar-Encoded OTLP

The current OpenTelemetry Protocol is an effective but straightfor‐ ward encoding of the OpenTelemetry data model. Spans, metrics, and logs are encoded as...spans, metrics, and logs. This can be thought of as a *row-based model*, where every span, metric, or log is encoded as a discrete element in a list of elements. A column-based model would instead encode every element into a single table, where all elements share the same columns for overlapping concepts such as timestamps and attributes.

This columnar representation is expected to optimize the creation, size, and processing of data batches. The main benefits of a such an approach are:

- Better data compression rate (group of similar data)

- Faster data processing (better data locality means better use of the CPU cache lines)

- Faster serialization and deserialization (fewer objects to process)

- Faster batch creation (less memory allocation)

- Better I/O efficiency (less data to transmit)

The benefit of this approach increases proportionally with the size of the batches. Using the existing “row-oriented” representation is well suited for small batch scenarios. Therefore, columnar encoding would *extend* the current protocol. The current implementation being proposed is based on Apache Arrow, a well-established col‐ umnar memory format.

This optimization is focused on allowing high-volume data sources, such as content delivery networks and large multitenant systems, to participate in observability much like regular services. Columnar encoding will also reduce costs for egressing telemetry across network boundaries.
