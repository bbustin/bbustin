+++
title = "My self-taught journey to becoming a Software Engineer"

[taxonomies]
tags = ["Mentoring", "Career"]
+++

I hold an undergraduate degree, but not in Computer Science. Ever since the first
time I saw a computer, I've been fascinated and obsessed. My curiosity is what
ultimately led to my career as a Software Engineer.
<!--more-->

In elementary school, I was coding in [Applesoft BASIC](https://en.wikipedia.org/wiki/Applesoft_BASIC)
on the classroom computers. Even this simple programming language allowed me to do all sorts
of neat things. I drew on graph paper and then programmed the computer to display it by entering
the coordinates. I learned about conditional logic. It became apparent to me that there
was so much I could use it for.

Fast forwarding a ton, in my first job after college (that was not my own IT consulting firm), I was in
charge of running a computer lab for [ESOL](https://en.wikipedia.org/wiki/English_as_a_second_or_foreign_language)
students at the local community college. There were so many repetitive tasks. I don't know
about you, but repetition to me is like a flashing red sign that says "Don't waste, automate!".
I taught myself Python and automated away a lot of my work.

Fast forward a lot more. I'm in a role in the Citrix Education Department where I create
lab manuals and virtualized environments that go with them. This is to support training
courses for candidates wanting to get Citrix Certified. The virtualized lab environments
are real Windows virtual machines with Citrix products installed. They need to be set up
such that the steps in the lab guides always work.

I am creating the lab environments and manuals in lockstep and have a huge pain point.
It is hard to iterate when it takes a day for changes to my lab environment to be
captured. I can't test my changes in the golden image because any changes I make will
leave traces on the virtual machines. It also takes up to 8 hours for one of these lab
environments to be provisioned and at least 30 minutes for one to be reset.

What I need to be significantly more productive is a system that can quickly:

* capture lab environments
* deploy lab environments

I whipped out my Python skills and created a novel proof of concept (PoC). My co-worker
saw the PoC and immediately knew it had great potential. He and I brainstormed and
together we created a more polished PoC which we presented to upper management. They
were gobsmacked. How could two courseware developers have created a lab capture
and provisioning system that rivals the third party one currently in use?

My co-worker and I continued to develop this new system in our spare time. The company
thought "there is no way these two guys can create a world-class lab delivery system",
but they did see how much money could be saved by a system that could provision these
lab environments so quickly. It would mean the training labs could be delivered on demand.
They would not need to schedule it all in advance which inevitably leaves costly resources idling. They
issued an RFP to several large IT consultancies, including [Infosys](https://www.infosys.com/).

We participated in many meetings where these companies presented their proposals. Not
one of the proposals came close to the functionality and resilience our system provided.
The department decided to trial our system with some training that was being delivered in-house.
It worked great.

Word of this new system spread throughout the company. The sales organization had training
labs at the company's two flagship conferences Citrix Synergy and Citrix Summit. The training
labs were self-service. Conference attendees could just walk up, sit at a computer, and choose
from one of several training labs to complete. To support this, the team had to try to forecast
demand for each lab the night before. They would then provision the resources proportionally.
They also had to order a lot more resources than were ultimately needed because it took
quite some time to reset.

We used the new provisioning system at the next event and they were hooked. We had slashed
the costs and were delivering a better end-user experience. Performance was much faster
than it had previously been and we no longer ran out of capacity for one or more of the
labs. With many more successful events under my co-worker and my belts, we were both
moved into the sales organization.

My new position was to work on the backend of the [Citrix Demo Center](https://demo.citrix.com).
This meant programming in Java. I had never touched any Java code at that point. I
buckled down and taught myself Java. Within two weeks I was already contributing code.
It wasn't great code, mind you, but I still think being able to pull that off is pretty
impressive. It was also fun. My co-worker was moved to an adjacent project.

Over time I worked on lots more. Including new provisioning types on Google Cloud Platform,
Amazon Web Services, Microsoft Azure, and Citrix Cloud. Later on I designed a replacement
for the legacy system that managed all of this provisioning. I planned how we were going to
transition off of the old system without the users even noticing. I led the project, trained
a new engineer, and contributed lots of code myself. My co-worker from the education days
was pulled onto this project and, for a while, we got to re-live the amazing collaboration
of our early years at Citrix.

The legacy system has since been retired. The new system has significantly less technical debt.
I designed it defensively so that it should accrue this debt at a much slower rate. It is
also much easier to maintain because the design makes it nearly impossible for changes to
one type of provisioning form having any effect on the other types of provisioning. I call
the core idea behind the design a "modular monolith". It delivers the isolation microservices
have without the deployment complexity and many failure modes.

That is how I got to where I am now.
