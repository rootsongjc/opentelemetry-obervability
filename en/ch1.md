# Myths and Historical Accidents

The world of observability is in the midst of a seismic shift.

Traditionally, we have observed our systems using a set of siloed, independent tools, most of which contained poorly structured (or completely unstructured) data. These independent tools were also vertically integrated. The instrumentation, protocols, and data for‐ mats all belonged to a particular backend or service and were not interchangeable. This meant that replacing or adopting new tools required the time-consuming work of replacing the entire toolchain, not just switching backends.

This siloed technology landscape is often referred to as the “three pillars” of observability: logging, metrics, and (almost never) tracing (see Figure 1-1).

*Logging*

Recording the individual events that make up a transaction.

*Metrics*

Recording aggregates of events that make up a transaction.

*Tracing*

Measuring the latency of operations and identifying perfor‐ mance bottlenecks in a transaction—or something like that. Traditionally, many organizations do not make use of dis‐ tributed tracing, and many developers are unfamiliar with it.

*Figure 1-1.* *The* *“three pillars” of observability.*

We’ve worked with this approach for so long that we don’t often question it. But as we’ll see, the “three pillars” is not a properly structured approach to observability. In fact, the term only describes how certain technologies happened to have been implemented, and it obscures several fundamental truths about how we actually use our tools.

Transactions and Resources: What It’s

All About

Before we dive into the pros and cons of different observability paradigms, it’s important to define what it is we are observing. The distributed systems that we are most interested in are internet-based services. These systems can be broken down into two fundamental components: *transactions* and *resources*.

A *transaction* represents all of the actions a distributed system needs to execute in order for the service to do something useful. Loading a web page, purchasing a shopping cart full of items, ordering a ride share—these are all examples of transactions. It is important to understand that transactions are not *just* database transactions. Every request to every service is part of the transaction.

For example, a transaction may start with a browser client making an HTTP request to a proxy. The proxy first makes a request to an authentication system to verify the user, then forwards the request on to a frontend application server. The application server makes several requests to various databases and backend systems. A mes‐ saging system—Kafka or AMQP, for example—is used to queue up additional work to be processed asynchronously. All of this work must be done correctly to deliver a result to the user, who is waiting impatiently. If any portion fails or takes too long, the result is a bad experience, so we need to understand transactions in their entirety.

Along the way, all of these transactions use up *resources*. Web serv‐ ices can only handle so many concurrent requests before their performance degrades and they begin to fail. These services can be scaled up, but they interact with databases that may invoke locks, creating bottlenecks. Database requests to read a record may block requests to update that record, and vice versa. Furthermore, all of these resources cost money, delivered as a bill from your infrastruc‐ ture provider at the end of each month. The more you consume, the higher your bill.

How do you fix a problem or improve the quality of a service? Either a developer modifies the transactions, or an operator modi‐ fies the available resources. That’s it. That’s all there is to it. The devil, of course, is in the details.

An ideal way for observability tools to represent transactions and resources is covered in detail in Chapter 4. But let’s get back to describing the non-ideal way we have been doing it to date.

> **Okay, Okay, Not *Everything* Is a Transaction**
>
> Note that there are additional computational models beyond trans‐ actions. For example, desktop and mobile applications are often based on a *reactor* model, where the user interacts continuously for a long period of time. Photo editing and video games are good examples of applications that have long user sessions that can be hard to describe as discrete transactions. In these cases, event-based observability tools such as *real user monitoring* (RUM) can augment distributed tracing to better describe these long-running user sessions.
>
> That said, almost all internet-based services are built upon a trans‐ actional model. Since these are the services we are focusing on, observing transactions is what we describe in this report. RUM is described in more detail in Appendix B.

The “Three Browser Tabs” of Observability

With transactions and resources in mind, let’s have a look at the three pillars model. Labeling this separation of concerns as “three pillars” makes it sound intentional—wise, even. Pillars sound majes‐ tic and serious, like the ancient Parthenon of Athens. But this arrangement is actually a rather unfortunate accident. The real rea‐ son that computer observability consists of multiple siloed, vertically integrated toolchains is simply a banal story of historical convenience.

Here’s an example. Say that one person wants to understand the transactions that their program is executing. So they build a logging tool: an interface for recording messages that contain timestamps, a protocol for sending those messages somewhere, and a data system for storing and retrieving them. Simple enough.

Another person wants to monitor all of the resources being used at any given moment; they want to capture metrics. Well, they wouldn’t use a logging system for that, would they? They want to track how a value changes over time and across a limited set of dimensions. Clearly, a giant pile of unstructured log messages has little to do with this problem. So a new and entirely separate system is created, solv‐ ing the specific problem of generating, transmitting, and storing metrics.

Another person wants to identify performance bottlenecks. Again, the unstructured nature of the logging system renders it irrelevant. Identifying a performance bottleneck, such as a sequence of opera‐ tions that could instead be run in parallel, requires that we know the duration of every operation in a transaction and how those opera‐ tions are linked together. So an entirely separate tracing system is built. And since the new tracing system is not intended to replace the existing logging system, the tracing system is heavily sampled to limit the cost of running a third observability system in production.

This piecemeal approach to observability is a completely natural and understandable process of human engineering. However, it has its limitations, and unfortunately those limitations can often be at cross-purposes to how we use (and manage) these systems in the real world.

How We Actually Observe Our Systems in the

Real World

As a refresher, let’s go through the actual, unvarnished process that operators go through when they investigate a problem.

Investigation involves two steps: noticing that something has hap‐ pened, and then determining what caused it to happen.

When we perform these tasks, we use our observability tools. But, importantly, we don’t use each tool in isolation. We use all of them together. And all along the way, the siloed nature of these tools puts an enormous cognitive load on the operator.

Often, the investigation starts when someone notices that an impor‐ tant metric has *gone all squiggly*. In this case, “squiggly” is an important term of art, as often the only information an operator has at this point is the shape of a tiny line on a dashboard and their own internal assessment as to whether the shape of that line looks “right” or not (Figure 1-2).

*Figure 1-2. Traditionally,* *finding* *correlations across* *different* *data sets is a terrible experience.*

Having determined the point where the shape of the line started to look “off,” the operator would then squint and try to find other lines on the dashboard that went “squiggly” at the same time. However, because these metrics are completely independent of each other, the operator must do the comparison in their brain, without help from the computer.

Needless to say, staring at charts in hope of finding a useful correla‐ tion takes time and brainpower, not to mention it leads to eyestrain.

Personally, I’d use a ruler or a piece of paper and just look at what lined up. In “modern” dashboards, the ruler is now a line drawn as part of the UI. But this is just as crude a solution. The real work of identifying correlations must still occur in the operator’s head, again with no help from the computer.

Having gained an initial, rough guess about the problem, the opera‐ tor usually begins to investigate transactions (logs) and resources (machines, processes, configuration files) they believe may be asso‐ ciated with the problem (Figure 1-3).

*Figure 1-3. Finding the logs associated with an anomaly is also a terrible experience.*

Here, again, the computer is no real help. The logs are stored in a completely separate system and cannot be automatically associated with any metrics dashboard. Configuration files and other service- specific information are often in *no system*, and operators must SSH or otherwise access running machines to look at them.

Therefore, the operator is once again left to do the job of finding correlations, this time between metrics and the relevant logs. Identi‐ fying these logs can be difficult; often the source code must be consulted to even get an idea of what logs might be present.

When a (possibly, hopefully) relevant log is found, the next step is usually to determine the chain of events that caused this log to be generated. This means finding the rest of the logs in the same transaction.

Once again, a lack of correlations puts a huge burden on the opera‐ tor. Unstructured and semistructured logging systems do not have a mechanism for automatically indexing and filtering logs by transac‐ tion. Even though this is by far the most common logging workflow that operators perform, they are left to perform an ad hoc series of queries and filters to winnow down the available logs into a subset that, hopefully, represents an approximation of the transaction. To even have a chance of success, they must lean on application devel‐ opers to add various request IDs and other breadcrumbs to find later and stitch together.

In a small system, this process of reconstructing transactions is tedious but possible. But once a system grows to include many hori‐ zontally scaled services, the amount of time it takes to reconstruct a single transaction begins to seriously limit the scope of any investi‐ gation. Figure 1-4 shows a complex transaction that touches many services. How would you collect all of the logs?

*Figure 1-4. With traditional logging,* *finding* *the exact logs that make up a* *specific* *transaction can take a lot of* *effort.* *Once systems become large enough, it becomes almost impossible.*

One great answer is *distributed tracing,* which actually has all of the IDs and indexing tools needed to reconstruct a transaction automat‐ ically. Unfortunately, tracing systems are often seen as a niche tool for latency analysis. As a result, they are sent comparatively little logging data. And because they are focused on latency analysis, trac‐ ing systems are often heavily sampled, rendering them irrelevant for these kinds of investigations.

Not Three Pillars, but a Single Braid

Needless to say, this is a bummer. The above workflow really does represent a terrible state of affairs. But because we’ve been living with this technology regime for so long, we often don’t recognize how inefficient it actually is, compared to what it could be.

Today, to understand how their systems are changing, operators must first gather large amounts of data. Then they must use their minds to identify correlations in that data, based on visual stimuli such as dashboard displays and log scanning. This is an intense mental effort. It also would be unnecessary, if a computer program could scan and correlate this data automatically. Operators would save time—often in situations where time is critical—if they could focus on investigating *how* their system is changing, without first having to identify *what* is changing.

Before writing a computer program that could accurately perform this kind of change analysis, all of these data points would need to be connected. Logs would need to be linked together so that the transactions could be identified. Metrics would need to be linked to logs so that the statistics being generated could be connected to the transactions they are measuring. And each data point would need to be linked to the underlying system resources—software, infrastruc‐ ture, and configuration details—so that all events could be connec‐ ted to a topology of the entire system.

The end result—a single, traversable graph containing all of the data needed to describe the state of a distributed system—is the type of data structure that would give an analysis tool a complete view of the system. Rather than “three pillars” of disconnected data, we would have a single braid of interconnected data.

This is where OpenTelemetry comes in. As illustrated in Figure 1-5, OpenTelemetry is a new telemetry system that generates traces, logs, and metrics in an integrated fashion. All of these connected data points are then transmitted together in the same protocol, which can then be input into a computer program for identifying correla‐ tions across the entire data set.

*Figure 1-5. OpenTelemetry takes all of this siloed information and connects it as a single, highly structured data stream.*

What does this unified data look like? In the next chapter, we’ll set the three pillars aside and build a new mental model from scratch.
