# Stability and Long-Term Support

OpenTelemetry was designed to allow long-term stability and inno‐ vation to occur side by side. In OpenTelemetry, stability guarantees are provided on a signal-by-signal basis. Rather than look at the ver‐ sion number, check the stability rating for the signals you wish to use.

## Signal Lifecycle

Figure 6-1 shows how new signals are added to OpenTelemetry. *Experimental* signals are still under development. They may change —and break compatibility—at any time. Experimental signals start development as a specification proposal, which is developed along‐ side a set of prototypes. Once an experimental signal is ready to be used in production, a feature freeze is put on the signal’s specifica‐ tion, and beta releases of the new signal are created in multiple languages. Beta releases may not be feature complete, and they may experience breaking changes, but they are considered ready for pro‐ duction feedback from early adopters. Once a signal is considered ready to be declared *stable*, a release candidate is issued. If the release candidate can remain stable and without issue for a period of time, then both the specification and the beta releases of the signal are declared stable.

Once a signal becomes stable, it falls under OpenTelemetry’s long-term support guarantees. OpenTelemetry takes backward com‐ patibility and seamless upgrades very seriously. See the following sections for details.

Should a component of an OpenTelemetry signal need to be retired, that component will be marked as *deprecated*. Deprecated compo‐ nents no longer receive new features, but they are still covered by OpenTelemetry’s long-term support guarantees. If possible, the component will never be removed and will continue to function. If a component must be removed, a removal date will be declared well in advance.

Experimental features are always kept in separate packages from sta‐ ble features, and stable features may never reference experimental features. This ensures that new developments never affect the stabil‐ ity of existing features. As long as libraries only depend on stable features, they will never experience a breaking API change.

*Figure 6-1. Every major feature in OpenTelemetry is assigned a stability rating and follows the same lifecycle.*

## API Stability

The OpenTelemetry API is expected to be depended upon by thou‐ sands of libraries, with millions of call sites. As such, stable portions of the API must never break backward compatibility. Application owners and library developers should never have to re-instrument their application in order to upgrade to a new version of the API.

If an OpenTelemetry API were to be deprecated, which is unlikely, the deprecated API would still remain stable and functional.

Native Instrumentation is Stable Instrumentation

OSS libraries that ship with native OpenTelemetry instrumentation should only use stable APIs, as changes to experimental features may create a dependency conflict.

That said, we definitely encourage beta testing. If a library would like to offer support for experimental OpenTelemetry features, that is a great way to give the project feedback and participate in the design of new features. However, we recommend that integration with experimental OpenTelemetry features be offered as optional plug-ins, which the end user must install separately to enable.

Once features become stable, there is no need to keep them as a separate plug-in. In fact, it is better to integrate OpenTelemetry natively, as users may forget to install plug-ins. That way, if the application owner installs the OpenTelemetry SDK, they automati‐ cally start receiving data from every library. If no SDK is installed, the API calls are no-ops. Native instrumentation is one way we hope that OpenTelemetry will simplify observability for application owners—it’s already present in every library and ready to go as soon as it’s wanted.

SDK and Collector Stability

SDK stability focuses on two areas: plug-in interfaces and resource usage. The SDK may occasionally deprecate a plug-in interface. To ensure that application owners can upgrade cleanly, a replacement interface must be added before the deprecated interface is removed, and popular plug-ins that use the deprecated interface must be migrated to the new interface. By safely migrating the plug-in eco‐ system in this manner, we avoid catching application owners in a situation where they want to upgrade, but are blocked by an incom‐ patible plug-in. There must be at least a six-month window between deprecating and removing a plug-in interface, and deprecated inter‐ faces should only be slated for removal when maintaining them creates a performance problem. Otherwise, we retain the deprecated interfaces indefinitely.

Speaking of performance, stable portions of the OpenTelemetry SDK must avoid performance regressions, to ensure that newer ver‐ sions of the SDK do not induce resource contention when they are upgraded. Obviously, enabling additional features added in a newer version may require additional resources. But simply upgrading the SDK should not cause a performance regression.

Much like the SDK, the Collector seeks to avoid performance regressions and provide a progressive upgrade path for the Collector plug-in ecosystem.

Upgrading OpenTelemetry Clients

When running OpenTelemetry, it is expected that users will stay up-to-date with the latest version of the SDK. Two events may force a user to upgrade the SDK: a library upgrades its instrumentation to a new version of the API, or OpenTelemetry issues a critical security patch.

The stability guarantees listed above ensure that this upgrade path is always feasible. As long as an application only depends upon stable signals, upgrading should only involve a dependency bump. Instru‐ mentation will not need to be rewritten, and plug-ins will not suddenly become unsupported.

A good example of OpenTelemetry’s commitment to backward com‐ patibility can be found in its support for its predecessor, OpenTracing. The OpenTelemetry tracing signal remains fully com‐ patible with the OpenTracing API, and both OpenTelemetry and OpenTracing API calls can be mixed into the same application. OpenTracing users can upgrade to OpenTelemetry without needing to ever rewrite existing instrumentation.