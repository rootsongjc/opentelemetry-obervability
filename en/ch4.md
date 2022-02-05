# Supporting Open Source and Native Instrumentation

So far we’ve discussed modern observability from the perspective of data. But there is another aspect to modern observability, that, in the long run, may prove equally important: how we approach the instrumentation that creates the data.

Most software systems are built with off-the-shelf parts: web frameworks, databases, HTTP clients, proxies, programming languages. In most organizations, very little of this software infrastructure is written in-house. Instead, these components are shared across many organizations. Most commonly, these shared components come in the form of open source (OSS) libraries with permissive licenses.

Since these OSS libraries encapsulate almost all of the critical func‐ tionality present in an average system, access to high-quality instru‐ mentation for these libraries is critical to most observability systems.

Traditionally, instrumentation is “sold separately.” This means that software libraries do not include instrumentation to produce traces, logs, or metrics. Instead, solution-specific instrumentation is added after the fact as part of deploying an observability system.

> **What is Solution-Specific Instrumentation?**
>
> In this chapter, the term *solution-specific* *instrumentation* refers to any instrumentation designed to work with a specific observability system, using a client developed as an outgrowth of that particular system’s data storage system. In these systems, the clients and stor‐ age systems often become deeply intertwined. As a result, a com‐ plete re-instrumentation is usually required for an application to switch from one observability system to another.

Solution-specific instrumentation is a holdover from the vertical integration inherent to the “three pillars.” Every backend ingests a specific type of proprietary data; ergo, the creators of those backends must also supply the instrumentation that generates that data.

This approach to instrumentation creates trouble for everyone involved in software development: the providers, the users, and the OSS library authors.

Observability Is Drowning in

Solution-Specific Instrumentation

From the perspective of an observability system, instrumentation represents an immense amount of overhead.

In the past, internet applications were fairly homogenous, and it was possible to center an observability system around a particular web framework: Java Spring, Ruby on Rails, or .NET. But over time, the diversity of software has exploded. Maintaining instrumentation for every popular web framework and database client now represents an immense investment.

The duplication of effort that results is simply enormous. Tradition‐ ally, vendors used their investment in instrumentation as both a sell‐ ing point and a form of gatekeeping. But the increasing rate of software development is beginning to make this untenable. The sur‐ face area is simply too great for a reasonably sized instrumentation team to keep up with.

This burden is especially grim for new, novel observability systems, especially OSS observability projects. If a novel system cannot be deployed in production until a significant amount of instrumenta‐ tion is written, and an OSS project cannot attract significant interest until it is widely deployable, then scientific progress will find itself at an impasse. This is especially true for tracing-based systems, which require end-to-end instrumentation to provide maximum value.

Applications Are Locked In by

Solution-Specific Instrumentation

From the perspective of an application developer, solution-specific instrumentation represents a pernicious form of lock-in.

Observability is a cross-cutting concern. To thoroughly trace, log, or metricize a large application means that thousands of instrumenta‐ tion API calls will be spread throughout the codebase. Changing observability systems requires ripping out all of this instrumentation and replacing it with different instrumentation provided by the new system.

Replacing instrumentation represents a significant up-front invest‐ ment, even just to try out a new system. Worse, most systems are large enough that a simultaneous swapping of all instrumentation across all services is infeasible. Most systems require a progressive rollout of new instrumentation, but executing such a rollout can be difficult to devise and involves even more effort.

Getting “trapped” by solution-specific instrumentation is incredibly frustrating. At the same moment that observability vendors are beginning to struggle with supplying their own instrumentation, users are beginning to refuse to adopt it. Knowing how much effort is involved in re-instrumentation, many users strongly prefer that any new observability system they are considering work with the instrumentation they are currently using.

To support this request, many observability systems attempt to work with instrumentation provided by several other systems. But this patchwork approach degrades the quality of the data each system is ingesting. Ingesting data from many sources means there is no longer a clear definition of the data being input, and it’s hard for analysis tools to do their jobs when the expected data is uneven and vaguely defined.

Solution-Specific Instrumentation for OSS

Is Basically Impossible

From the perspective of an open source library author, solution- specific instrumentation represents a tragedy.

Telemetry from OSS libraries is critical to operating the software applications built on top of them. The people with the greatest understanding of what data is critical to operations—and how oper‐ ators should leverage this data to remediate problems—are the OSS library developers who actually wrote the software.

But library authors are in a bind. As we’ll see, no solution-specific instrumentation API, no matter how well written, is an acceptable choice for OSS libraries.

How Do You Pick a Logging Library?

Let’s say that you’re writing the world’s greatest OSS web framework. Many things can go wrong during production, and naturally you would like to communicate errors, debugging, and performance information to your users. Which logging library do you use?

There are plenty of decent logging libraries. There are so many, in fact, that no matter which library you pick, you are guaranteed to have many users who wish you had picked a different one. What if your web framework picks one logging library and the database cli‐ ent library picks a different one? What if neither is the one the user wants to use? What if they choose incompatible versions of the same library?

There is no clean way to combine multiple solution-specific logging libraries into a coherent system. And while logs are simple enough that a hodgepodge of different solutions might be workable, that is not the case for solution-specific metrics and tracing.

As a result, OSS libraries do not usually ship with logging, metrics, or tracing built in. Instead, libraries settle on providing “observabil‐ ity hooks,” which require their users to write and maintain a pile of adapters to connect the libraries they use up to their observability system.

Authors have a wealth of knowledge they would like to communi‐ cate about how their systems should be run, and they have no clear way to do it. If you ask anyone who has written a significant amount of open source software, they will tell you: This situation is painful! And unfortunate! Some library authors *do* try to choose a logging library, only to discover that they have inadvertently created a ver‐ sion conflict for some of their users, while forcing other users to write logging adapters to capture the data in using their *actual* log‐ ging library.

But for most libraries, observability simply becomes an after‐ thought. While library authors often write extensive test suites, they rarely spend much time thinking about runtime observability. Given that libraries have plenty of testing tools but zero observability tools, this is hardly a surprising outcome.

As we’ll see in the next few chapters, modern observability is designed to maximize agency for everyone who has a role in the observability pipeline. But it is library authors who stand to benefit the most; with solution-specific instrumentation, they currently have no options at all.

Decomposing the Problem

We can solve all of the issues listed above by designing an observa‐ bility system to explicitly address the needs of everyone involved. For the rest of this chapter, we will break down the design of a modern observability system into fundamental requirements. These requirements will provide motivation for the architecture of OpenTelemetry described in Chapter 5.

Requirement: Separate Instrumentation, Telemetry, and Analysis

Ultimately, computer systems are actually human systems. A cross- cutting concern such as observability interacts with almost every single software component. At the same time, transmitting and processing telemetry can be such a high-volume activity that a large- scale observability system generates its own operational concerns. This means that many different people, serving in different capaci‐ ties, need to interact with different aspects of the observability sys‐ tem. To serve them well, this system must ensure that everyone involved has the agency they need to perform their tasks quickly and independently. Providing agency is a fundamental design require‐ ment for an effective observability system.

Let’s start by identifying the responsibilities of every role related to a running software system: the library author, the application owner, the operator, and the responder.

*Library authors communicate what their* *software* *is doing.*

For software libraries that encapsulate critical functionality, such as network and request management, library authors also must manage aspects of the tracing system: injection, extrac‐ tion, and context propagation.

*Application owners compose* *software* *and manage dependencies.*

Application owners choose the components that comprise their application and ensure that they compile into a coherent, func‐ tional system. Application owners also write application-level instrumentation, which must interact cleanly with the instru‐ mentation (and context propagation) provided by the library authors.

*Operators manage the production and transmission of telemetry.*

Operators manage the transmission of observability data from applications to responders. They must be able to choose what format the data is in and where it is being sent. While the data is in flight, they must operate the transmission system: managing all of the resources required to buffer, process, and route the data.

*Responders consume telemetry and generate useful insights.*

To do that, responders must understand both the structure and the meaning inherent to the data. (Structure and meaning are described in detail in Chapter 3.) Responders also need to add new and improved analysis tools to their toolbox when they become available.

These roles represent different decision points:

- Library authors can only make changes by releasing a new version of their code.
- Application owners can only make changes by deploying a new version of their executable.
- Operators can only make changes by managing the topology and configuration of executables.
- Responders can only make changes based on the data they are receiving.

The traditional three pillars approach scrambles all of these roles. A side effect of vertical integration is that almost any data change requires a code change. Almost any non-trivial change to the observability system requires application owners to make code changes. Requiring that other people make the changes you care about creates barriers to change and can lead to stress, conflict, and inaction.

Clearly, a well-designed observability system should focus on allow‐ ing everyone as much agency and direct control as possible, and it should avoid turning developers into accidental gatekeepers.

Requirement: Zero Dependencies

Applications are composed of dependencies (web frameworks, data‐ base clients), plus their dependencies’ dependencies (OpenTelemetry or other instrumentation libraries), plus their dependencies’ depen‐ dencies’ dependencies (whatever those instrumentation libraries depend on). These are called transitive dependencies.

If there is a conflict between any two dependencies, the application cannot function. For example, two libraries may each require a dif‐ ferent (and incompatible) version of a low-level networking library, such as gRPC. This can lead to a number of bad situations. For example, a new version of the library might include a needed secu‐ rity patch but also include an upgraded dependency, which creates a dependency conflict. Yikes!

Transitive dependency conflicts such as this create major headaches for application owners, as the conflicts cannot be solved independ‐ ently. Instead, application owners must contact the library authors and request that they provide a solution, which ends up taking time (assuming the library authors even answer the request).

For modern observability to work, libraries must be able to embed instrumentation without fear that it will lead to problems when their library is composed into an application. Therefore, an observability system must provide instrumentation that does not contain depen‐ dencies that may inadvertently trigger a transitive dependency conflict.

Requirement: Strict Backward Compatibility and Long-Term Support

When an instrumentation API breaks backward compatibility, bad things happen. A well-instrumented application may end up with many thousands of instrumentation call sites. Having to update thousands of call sites due to an API change is a significant amount of work.

This creates a particularly bad form of dependency conflict, in which instrumentation in one library is no longer compatible with instrumentation in another library.

Therefore, instrumentation APIs must be strictly backward compat‐ ible, over a very long time scale. Ideally, instrumentation APIs never break backward compatibility—ever—once they have become stable. And new, experimental API features must be developed in such a way that their existence does not create conflicts between libraries that contain stable instrumentation.

Separation of Concerns Is Fundamental to

Good Design

In the next chapter, we’ll dive into the architecture of OpenTelemetry and see how it meets the requirements laid out above. But before we do that, there is an important point I would like to make.

If you analyze these requirements, you may notice something peculiar: there is almost nothing about them that is specific to observability. Instead, the focus is on minimizing dependencies, maintaining backward compatibility, and ensuring that different users can perform their roles without needless interference.

Each requirement points to a separation of concerns as a key design feature. But these features are not unique to OpenTelemetry. Any software library that seeks wide adoption would do well to include them. In that sense, the design of OpenTelemetry can also be a guide to designing open source software in general. The next time you start a new OSS project, consider these requirements up front and design your libraries accordingly—your users will thank you!