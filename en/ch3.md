# The Limitations of Automated Analysis

Automation is beginning to sound amazing, but here’s an important reality check: at the end of the day, computer analysis will not be able to tell you what is wrong with your system or fix it for you. It can only save you time.

With that in mind, I want to stop with the praise for a moment and acknowledge some limitations of automation. I do this because there will be quite a lot of hype around combining artificial intelligence and observability. This kind of hype leads to amazing claims along the lines of “AI anomaly detection and root-cause analysis will fully diagnose your problems!” and “AI operations will fully manage your system!”

## Beware of Hype

To derail this hype train, I want to be clear that these kinds of dreamy marketing claims are *not* what I am claiming modern observability will provide. In fact, I predict that many claims related to intelligent AI problem-solving will mostly be a lot of snake oil.

Why won’t AI solve our problems? In general, machines cannot identify “problems” in software because defining what counts as a “problem” requires a form of subjective analysis. And the kind of real-world AI we are talking about cannot perform subjective decision making with any degree of accuracy. Machines will always lack sufficient context.

For example, let’s say that a release slows down performance. But it also contains a really important feature that’s number one on every user’s wish list! In that case, the performance regression is a feature, not a bug.

While they might seem like similar activities, there is a vast chasm between *identifying correlations* and *identifying a root cause*. When you have the right data structures, correlations are a kind of *objective analysis—*you just have to crunch the numbers. Which of those cor‐ relations are relevant and indicate the source of a real problem will always require *subjective analysis,* an interpretation of the data. What do all these correlations mean? As Jeffrey Lebowski would say, “Well, you know, that’s just, like, your opinion, man.”

### Magical AIOps

When investigating a system, two types of analysis come into play:

- Objective analysis, which is fact based, measurable, and observable
- Subjective analysis, which is based on interpretations, points of view, and judgment

This dichotomy—objective versus subjective—relates to an impor‐ tant problem in computability theory called the *halting problem*. The halting problem is defined as whether a computer program could be written that could determine whether any arbitrary pro‐ gram would finish running or continue to run forever, given a description of that arbitrary computer program and its input. The short version is that in 1936, Alan Turing proved that a general algorithm to solve the halting problem cannot exist, and an exten‐ sion of this proof can be applied to many forms of identifying “problems” in computer software.

What Alan Turing is saying is that we don’t get to have magical AIOps (artificial intelligence for IT operations). Look for tools that promise to automate tedious objective analysis—crunching num‐ bers is what machines are good for! But beware of tools that claim to find the root cause of problems and automatically fix them; they will most likely leave you disappointed.

Here’s why we can identify correlations but not causation: imagine a machine that could determine what constitutes problematic behav‐ ior in any arbitrary computer program and identify the root cause of that behavior. If we actually built such a machine, it would be time to break out the champagne, because it would mean that we have finally solved the halting problem! However, there is no sign that machine learning has moved beyond Alan Turing’s model of univer‐ sal computing; you can trust that this won’t be happening.

Making the right call and shipping the fix is still on you.

## Time Is Our Most Precious Resource

However, we don’t need magical AIOps to see a huge improvement in our workflow. Identifying correlations while fetching relevant information so that you can browse through it effectively is some‐ thing that computers can *absolutely* do for you! And that will save you time. A lot of time. So much time that it will fundamentally change the way you investigate your systems.

Experiencing a massive reduction in wasted time is at the heart of what it means to practice modern observability. Crunching the numbers to identify correlations can be difficult even in simple sys‐ tems—at scale it is almost impossible. By shifting the cognitive burden onto machines, operators become able to effectively manage systems that have grown beyond what any human can possibly fit in their head.

But this shift in how we analyze telemetry is not the only major change that is about to take place in the world of observability. Much of the telemetry we need comes from software we didn’t write: the open source libraries, databases, and managed services we rely upon. These systems have traditionally struggled with instrumenta‐ tion and producing telemetry. What data we have access to, and where it comes from, is also about to change in a fundamental way.