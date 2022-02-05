# Foreword

Software development has changed (yet again). Open source soft‐ ware and public cloud providers have fundamentally shifted how we build and deploy software services. With open source software, we no longer need to write 100% of the code in our software applica‐ tions. And by using public cloud providers, we no longer need to provision servers or wire up network gear. All of this means that you can build and deploy an application—from scratch—in just days, if not hours.

But just because it’s easy to deploy new applications doesn’t mean that it’s gotten any easier to operate and maintain them. As applica‐ tions have become more complex and more heterogeneous—and most of all, more distributed—seeing the big picture, as well as pin‐ pointing exactly where a problem is occurring, has become more difficult.

But some things haven’t changed: as developers and operators, we still need to be able to understand application performance from a user’s point of view, and we still need an end-to-end view of user transactions. We also still need to measure and account for how resources are being consumed in the course of processing these transactions. That is, we still need *observability*.

But while the *need* for observability hasn’t changed, the way that we go about *implementing* observability solutions must change. This report details how traditional ways of thinking about observability are insufficient for modern applications: implementing observability as a collection of tools (in particular, as the “three pillars”) makes it almost impossible to operate modern applications in a reliable way.

OpenTelemetry addresses these challenges by providing an *integra‐ ted* approach to gathering data about application behavior and per‐ formance, including metrics, logs, and traces. As this report explains, OpenTelemetry is a new way to generate and collect this data, a way that is designed for cloud-native applications built with open source components.

In fact, OpenTelemetry is designed specifically with these open source components in mind. Unlike vendor-specific instrumenta‐ tion, OpenTelemetry can be embedded directly within open source code. This means that open source library authors can leverage their expertise to add high-quality instrumentation without adding any solution- or vendor-specific code to their projects.

OpenTelemetry has benefits for application owners, too. In the past, the way that telemetry was generated was tied to the way it was pro‐ cessed, stored, and analyzed. That meant that choosing which observability solution to use had to be done very early in the devel‐ opment process and was difficult to change afterward. OpenTeleme‐ try (like its predecessors, OpenTracing and OpenCensus) decouples the way that your application generates telemetry from how that telemetry is analyzed.

So often when we adopt new technology, we fail to consider how that technology will be used, and especially how it will be used dif‐ ferently by people in different roles. This report describes how OpenTelemetry meets the needs of library authors, application own‐ ers, operators, and responders and how it enables everyone in those roles to work independently—that is, to make key decisions autono‐ mously*—*as well as to collaborate effectively.

Perhaps most importantly, OpenTelemetry gives application owners and operators the flexibility to choose an observability solution that is best suited to their applications’ and their organizations’ needs, including the case where that requires multiple tools. It also includes cases where those needs change over time, and where new tools are required to satisfy those needs, as is likely the case as technology matures and organizations grow.

While users don’t need to commit to an observability solution, they do need to invest in a consistent way of generating telemetry. This report shows how OpenTelemetry is designed to be narrow in scope, extensible, and, most of all, stable: exactly what you’d expect if you were asked to make that investment.

being able to understand what’s happening in your application—are too high, and the costs of taking the wrong approach to observabil‐ ity can create debt that your organization will be paying back for years. Whether it’s an extended outage or just a development team that is bogged down in endless vendor migrations, the success of your organization depends on an integrated and open approach like OpenTelemetry.

This report offers practical advice about adopting and managing OpenTelemetry in your organization. It will get you started on the path toward a successful observability practice and unlock many of the true benefits of the cloud native and open source technologies: not only will you be able to build and deploy applications quickly, but you will be able to operate them reliably and with confidence.

*— Daniel “Spoons” Spoonhower Lightstep*
