# How to Roll Out OpenTelemetry Across Your Organization

Rolling out a new telemetry system can be a complex endeavor. It requires buy-in from across the engineering organization, and usu‐ ally cannot be deployed in a single step. Don’t underestimate the problems this may create!

In a large organization, there are usually many service teams that own different parts of the system. Often, each team will need to put in some amount of effort to get the services they manage fully instrumented. And those teams have their own backlog of work, which of course they want to prioritize.

An unfortunate result is that observability initiatives can run out of steam before they can begin to provide value and prove their worth. But with careful planning and coordination, this can be avoided.

The Primary Goal

When rolling out OpenTelemetry, it’s important to remember that any observability system based on distributed tracing will require every service participating in a transaction to be instrumented to provide maximum value. If only some of the services are instrumen‐ ted, the traces will be fragmented into small, disconnected pieces.

The results of this kind of scattershot instrumentation are unim‐ pressive. This situation—inconsistent instrumentation and broken traces—is the main thing you want to avoid. If the traces are disconnected, operators will still need to stitch everything together in their heads to get a picture of their system. Worse, automated analysis tools will have very limited data to work with. Unlike human operators, who can apply intuition and think outside the box, analysis tools are stuck with the data that they get. And with limited data, the chance that they will deliver useful insights will also be limited.

To avoid this trap, focus on instrumenting a single workflow, getting it completely traced end to end before moving on to the next work‐ flow. Most workflows do not touch every single part of a large system, so this approach will minimize the amount of effort needed before analysis can begin and value can be realized.

Pick a High-Value Target

Speaking of realizing value, when starting an instrumentation effort, it is important to have an attractive target!

Most likely, there is a particular issue motivating the drive to deploy OpenTelemetry. If that’s the case, focus on the minimal rollout needed to solve that issue. Otherwise, think about problems that are well-known enough that solving them would be worth making an announcement.

What painful, long-term issues are currently afflicting operations? Where would reducing system latency directly translate into busi‐ ness value? When people ask “why is it so slow?”, what part of the system are they talking about? Use this information as a guide when picking the first workflow you instrument.

Identifying an attractive target has two benefits. First, it can create buy-in because there’s a concrete reason to do the work. This makes it easier to convince the teams involved to prioritize adding instru‐ mentation and to do so in a coordinated fashion.

Second, a quick win gives your new observability system a chance to shine. The first time a software system is analyzed through the lens of distributed tracing, it almost always yields useful insights. Proving the value of observability can generate a lot of interest and helps to lower any lingering barriers to adoption.

If moving straight into production is difficult, “production support” systems are also a good place to start. CI/CD systems can be instrumented to help understand build and deployment perfor‐ mance. A big performance boost here will be felt across the organi‐ zation and can make a good case for moving OpenTelemetry into production.

Centralize Telemetry Management

Rolling out and managing a telemetry system benefits greatly from centralization. In some organizations, there is a platform or infra‐ structure team that has access to every service. A team such as this is a great place to centralize the management of telemetry, which can help immensely. The telemetry pipeline is best thought of as its own system; allowing one team to operate the entire telemetry pipeline is often better than asking many teams to each own a portion of the system.

On the software level, integrating OpenTelemetry setup with code management tools that are already widely deployed—shared init scripts and application frameworks, for example—reduces the amount of code each team needs to manage. This helps to ensure that services stay up-to-date with the latest versions and configura‐ tions and makes adoption much easier.

Another critical tool is a centralized repository of knowledge. OpenTelemetry has documentation, but it is generalized. Create documentation specific to deploying, managing, and using OpenTelemetry within your organization. It’s hard to overstress how much this helps, as most engineers will be new to OpenTelemetry and distributed tracing in general.

Breadth Before Depth

One more note on spending your effort wisely. When instrumenting a workflow end to end, it is usually sufficient to install the instru‐ mentation that comes with OpenTelemetry, plus instrumentation for any internal or homegrown frameworks used by your organization. It is not necessary to deeply instrument the application code, at least not at first. The standard spans and metrics that come with OpenTelemetry get you far enough to identify most problems; deeper instrumentation can be added selectively when necessary. Make sure you get end-to-end tracing up and running before adding this kind of detail.

That said, there is one quick way to add a lot of detail to your traces, which is to convert any existing logs into trace events. This can be done by creating a simple log appender that grabs the current span and attaches the log to it as an event. All of your application logs are now available when you look up the trace, which is far easier than finding them in a traditional logging tool. OpenTelemetry does pro‐ vide log appenders for some common logging systems, but they are also easy to write.

Work with Management

If you’re an engineer reading this, I have one additional note. Rolling out a new telemetry system can involve organizing a lot of people. Luckily, there are people who already do that kind of organ‐ izing—the managers!

I know, I know—managers. But if you’d like to start one of these ini‐ tiatives, convincing an engineering or project manager to help is a great first step. They will have valuable insight into how to get it done and will be able to pitch the project in meetings you may not attend. Sometimes organizing people is harder than organizing code, so don’t be afraid to ask for help!

Join the Community

Lastly, both on a personal and organizational level, consider joining the OpenTelemetry community! Maintainers and project leaders are friendly and very accessible. The community is an excellent resource for assistance and expertise; we are always happy to help new users get oriented. There is also a sweat equity culture: if there are Open‐ Telemetry features you’d like to see added, joining a working group and helping out is a great way to get them prioritized!

At minimum, be sure to give us feedback. The more we hear from users, the better we can focus on the issues that matter most. Our goal is to build the standard that drives the next generation of observability. We can’t do it without you, and our door is always open.

Thanks for Reading

I hope you’ve enjoyed this report on the future of observability with OpenTelemetry. If you have any questions, comments, feedback, or inspirations based on what you’ve read, please feel free to reach out to me on Twitter—I’m @tedsuo.

For help getting started with OpenTelemetry, please join the #OpenTelemetry channel on the Cloud Native Computing Founda‐ tion’s (CNCF) Slack instance. I hope to see you there!