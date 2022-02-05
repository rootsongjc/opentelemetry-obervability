# The Value of Structured Data

Squinting at graphs is not the best way to hunt for correlations. A huge amount of work currently done inside the heads of operators can actually be automated. This frees operators to move quickly among identifying issues, making hypotheses, and validating root causes.

To build better tools, we need better data. Telemetry must have two qualities to support high-quality automated analysis:

- All data points must be connected in a graph with proper indexing.
- All data points that represent common operations must have well-defined keys and values.

In this chapter, we will walk through a modern telemetry data model, starting with a basic building block: the attribute.

## Attributes: Defining Keys and Values

The most fundamental data structure is an *attribute,* which is defined as a key and a value. Every data structure in OpenTelemetry contains a list of attributes. Every component of a distributed system (an HTTP request, a SQL client, a serverless function, a Kubernetes Pod) is defined in the OpenTelemetry specification as a specific set of attributes. These definitions are called the OpenTelemetry *seman‐ tic conventions*. Table 2-1 shows a partial list of the HTTP conventions.

*Table 2-1. A partial list of HTTP conventions*

Having a standard schema such as this allows analysis tools to create detailed representations of the systems they are monitoring, while performing the kind of nuanced analysis that would be impossible when using poorly defined or inconsistent data.

## Events: The Basis for Everything

The most basic object in OpenTelemetry is an *event*. An event is simply a timestamp and a set of attributes. Using a set of attributes instead of a simple message/blog allows analysis tools to index events properly and make them searchable.

Some attributes will be unique to the event. Timestamps, messages, and exception details are all examples of attributes that are specific to a particular event.

However, most attributes are *not* unique to an individual event. Instead, they are common to a set of events. For example, the http.target attribute is relevant to every event recorded as part of an HTTP request. It would be inefficient to record these attributes over and over again on every event. Instead, we pull these attributes out into envelopes that surround the events, where they can be writ‐ ten once. Let’s call these envelopes *context*.

There are two types of context: static and dynamic (shown in

Figure 2-1). *Static context* defines the physical location where an event is taking place. In OpenTelemetry, these static attributes are called *resources*. Once the program starts, the values of these resource attributes usually do not change.

*Figure 2-1. While events have some* *event-specific* *attributes, most attributes belong to one of the contexts in which the event is occurring.*

*Dynamic context* defines the active operation in which the event is taking part. This operation-level context is called a *span*. Every time the operation executes, the values of these attributes change.

Not all events have both types of context. Free-floating events that only have resources, such as events emitted when a program is start‐ ing up, are called *logs*. Events that occur as part of a distributed transaction are called *span events*.

## Resources: Observing Services and Machines

Resources (static context) describe the physical and virtual infra‐ structure that a program is consuming. Services, containers, deployments, and regions are all resources. Figure 2-2 shows the resources involved in a typical shopping-cart checkout transaction.

*Figure 2-2. A transaction, viewed as a set of resources.*

Most problems in a running system stem from resource contention, many concurrent transactions attempting to make use of the same resources at the same time. By placing events within the context of the resources they are using, it becomes possible to automatically detect many types of resource contention.

Like events, resources can be defined as a set of attributes. Table 2-2 shows an example of a service resource.

*Table 2-2. Example of a service resource*

Besides the basic information needed to identify machines, configu‐ ration settings can also be recorded as resources. Having to access a running machine to understand how it is configured is a terrible burden. Instead, any important information found within a configu‐ ration file should also be represented as a resource.

## Spans: Observing Transactions

Spans (dynamic context) describe computer operations. A span has an operation name, a start time, a duration, and a set of attributes.

Standard operations are described using semantic conventions, such as the HTTP conventions described above. But there are also application-specific attributes, such as ProjectID and AccountID, which can be added by application developers.

Spans are also how we describe causality. To correctly record an entire transaction, we need to know which operations were triggered by which other operations. To do that, we need to add three more attributes to our spans: a TraceID, a SpanID, and a ParentID, as shown in Table 2-3.

*Table 2-3.* *Three* *additional attributes of spans*

These three attributes are fundamental to OpenTelemetry. By adding these attributes, all of our events can now be organized into a graph, representing their causal relationship. This graph can now be indexed in a variety of ways, which we will get to in a bit.

## Tracing: Like Logging, Only Better

We’ve now gone from simple events to events organized into a graph of operations associated with resources. This type of graph is called a *trace*. Figure 2-3 shows a common way for traces to be visualized, with a focus on identifying latency in operations.

*Figure 2-3. When logs are organized into a graph, they become traces.*

Essentially, tracing is just logging with better indexes. When you add the proper context to properly structured logs, you get traces almost by definition.

Think about how much time and effort you put into gathering those logs through searching and filtering; that is time spent gathering data, not time spent analyzing data. And the more logs you have to paw through—an ever-growing pile of machines executing an ever-increasing number of concurrent transactions—the harder it is to gather up that tiny sliver of logs that are actually relevant.

However, if you have a TraceID, gathering those logs is just one sin‐ gle lookup. Indexing by TraceID allows your storage tool to do this work for you automatically; you find one log and you have all the logs in the transaction right there, with no extra work.

Given that, why would you ever want “logs” without these “trace” IDs? We are so accustomed to the amount of work traditional log‐ ging forces us to do to connect the dots. But that work is not actually necessary; it is a by-product of the lack of structure in our data.

Therefore, distributed tracing isn’t just a tool for measuring latency; it’s a data structure for defining context and causality. It’s the glue that holds everything together. And as we’ll see, this glue includes the last remaining pillar—metrics.

## Metrics: Observing Events in Aggregate

Now that we’ve established what events are, let’s talk about events in aggregate. In an active system, the same events occur over and over, and we look at their attributes in aggregate to find patterns. The value of an attribute could occur too often, or not often enough, in which case we want to count how often these values are occurring. Or the value may exceed a certain threshold, in which case we want to gauge how the value is changing over time. Or we may want to look at the spread of values as a histogram.

These aggregate events are called *metrics*. And just like regular events, metrics have a set of attributes and a set of semantic conven‐ tions to describe common concepts. Table 2-4 shows some example attributes of system memory.

*Table 2-4. Attributes of system memory*

## Metrics Connected to Events: A Single

Unified System

Traditionally, we think of metrics as being completely separate from logs. But they are intimately connected. For example, let’s say that an API has a metric that measures the number of errors per minute. That’s a statistic. However, each error was created by a specific trans‐ action, using specific resources. Those specifics are present every time we increment that counter, and we want to know those specifics.

When an operator is alerted to a sudden spike in errors, their first question will naturally be “What’s causing that spike?” Looking at example traces may answer this question. Events that occurred ear‐ lier in the failed transaction (or that failed to occur) may be the source of the error.

In OpenTelemetry, when metric events occur within the context of a span, a sample of these traces is automatically associated with the metric as *trace exemplars*. This means that there’s no guessing or hunting for logs. OpenTelemetry definitively links traces and met‐ rics together. An analysis tool built on OpenTelemetry would let you go straight to your traces from your dashboards, with a single click. And if there is a pattern—for example, a particular attribute value correlates strongly with traces that result in a particular error—this pattern could be automatically identified.

## Automated Analysis and the Single Braid

Events, resources, spans, metrics, and traces: these are all connected in a single graph by OpenTelemetry, and they are all sent to the same database to be analyzed as a whole. This is the next generation of observability tools.

Modern observability will be built on data using structures that allow analysis tools to make correlations across all types of events and aggregates, and these correlations will have a profound effect on how we practice observability.

The transition to observing our systems holistically will have many benefits. But I believe that the primary time-saving features these new tools provide will be various forms of *automated correlation detection*. When searching for a root cause, noticing correlations can generate substantial insights. As shown in Figure 2-4, correlations are often the key ingredients of generating a root cause hypothesis, which can then be investigated further.

*Figure 2-4. Examples of correlations that may provide a key insight, pointing to a root cause.*

What does average performance look like? What do outliers look like? What trends shift together, and what do they have in common? Correlations may occur in many places: between attributes in a span, between spans in a trace, between traces and resources, within metrics, as well as in all those places together. When all of this data is connected in a graph, these correlations are ready to be discovered.

This is why a single braid of data is so critical. The value of any auto‐ mated analysis hinges entirely on the structure and quality of the data being analyzed. Machines traverse graphs of data; they don’t make leaps of logic. Accurate statistical analysis requires a telemetry system that is intentionally designed to support it.

## The Point: Automated Analysis Saves You Time

Why do we care about automating correlation analysis? Because time and complexity are against us. As systems grow in size, they eventually become too complex for any operator to hold entirely in their head, and there is never enough time to investigate every pos‐ sible connection when building a hypothesis.

The problem is that choosing what to investigate requires intuition, and intuition often requires a deep knowledge of each component that makes up a distributed system. As organizations grow their sys‐ tems and scale their engineering workforce accordingly, the portion of each system that any individual engineer deeply understands nat‐ urally shrinks to a smaller percentage of the entire system. Intuition doesn’t scale very well.

Intuition is also notoriously misguided; problems frequently arise in unexpected places. Almost by definition, problems that can be anticipated do not occur very often. What is left are all of the unan‐ ticipated problems, problems that have already slipped past our intuition.

This is where automated correlation detection comes in. With the right data, relevant correlations can be detected more effectively by machines. This allows human operators to move quickly, repeatedly testing hypotheses until they know enough to develop a solution.