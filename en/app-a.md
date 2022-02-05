# APPENDIX A: OpenTelemetry Project Organization

OpenTelemetry is a large project. Work on the OpenTelemetry project is divided into *special interest groups* (SIGs). While all project decisions are ultimately made via GitHub issues and pull requests, SIG members frequently stay in touch through the CNCF’s official Slack instance, and most SIGs meet once a week on Zoom.

Anyone can join a SIG. To find out more details about current SIGs, project membership, and project bylaws, check out the OpenTele‐ metry community repo on GitHub.

The Specification

OpenTelemetry is a specification-driven project. The OpenTeleme‐ try Technical Committee maintains the specification and guides the project forward by managing the specification backlog.

Small changes may be raised as GitHub issues, with the subsequent pull requests submitted directly to the specification. But significant changes to the specification occur via a request for comment process called OpenTelemetry Enhancement Proposals (OTEPs).

Anyone can submit an OTEP. The OTEPs are reviewed by specifica‐ tion approvers, who are grouped according to their area of expertise and assigned by the Technical Committee. OTEPs require at least four approvals to be accepted. A detailed design, along with proto‐ types in at least two languages, is usually required before any approvals can be made. OTEP authors are expected to take the requirements and concerns of other community members seriously. The goal is to ensure that OpenTelemetry fits the needs of as broad an audience as possible.

Once accepted, spec changes are drafted based on the OTEP. Spec changes only require two approvals, as most issues have already been resolved during the OTEP process.

Project Governance

The rules and organizational structures, which govern how the OpenTelemetry project functions, are defined and maintained by the OpenTelemetry Governance Committee, members of which are elected for two-year terms.

Governance members are expected to participate as individuals, not company representatives. But there is an upper limit to the number of committee members who may work for the same employer. If that threshold is exceeded because a committee member has changed jobs, committee members must resign until employer rep‐ resentation drops below the threshold.

Distros

OpenTelemetry has a plug-in-based architecture, since some observ‐ ability systems require a set of plug-ins and configurations to function correctly.

*Distros* (distributions) are defined as a collection of widely available OpenTelemetry plug-ins, plus a set of scripts or helper functions that may make it simpler to connect OpenTelemetry to a particular backend or run OpenTelemetry in a particular environment.

It is important to clarify that, for an observability system to claim it is compatible with OpenTelemetry, it should always be possible to use OpenTelemetry without being required to use a particular distro. If a project extends the core functionality of OpenTelemetry without going through the specification process, or includes any changes that cause it to become incompatible with the upstream OpenTelemetry repos, then that project is a fork, not a distro.

The Registry

To make it easy to discover what languages, plug-ins, and instru‐ mentation are currently available, OpenTelemetry provides a registry. Anyone can submit a plug-in to the OpenTelemetry regis‐ try; hosting the plug-in within the OpenTelemetry GitHub organiza‐ tion is not a requirement.
