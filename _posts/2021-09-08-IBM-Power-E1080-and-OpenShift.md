---
layout: post
title: IBM announces a Power E1080 beast, but OpenShift may steal the spotlight
---

On schedule, IBM launched its next-gen big-iron Power E1080 server ideal for huge databases. Included in the announcement is a peek at the future of Red Hat integration: a unique-to-IBM licensing option for OpenShift.

Today, September 8, IBM [announced](https://newsroom.ibm.com/2021-09-08-IBM-unveils-new-generation-of-IBM-Power-servers-for-frictionless,-scalable-hybrid-cloud) the Power E1080 server, an up-to-16 socket system that's the first to use its POWER10 processor. 

IBM refreshes its Power processor every 3 to 4 years, so this announcement wasn't unexpected. IBM also talked about its POWER10 processor a lot [last year](https://www.hc32.hotchips.org/assets/program/conference/day1/HotChips2020_Server_Processors_IBM_Starke_POWER10_v33.pdf), so even though the technical specs are impresive, they're not a surprise:

* 18 billion transistors using 7nm technology
* up to 15 cores per chip, with up to 16 chips per server
* Up to 8 threads per core ("SMT8"; similar to hyperthreading but with 8 instead of 2 threads per core)
* A new memory interface, new AI-enabling math accelerators, and a new built-in memory encryption feature
* Support for IBM's AIX and i operating systems, as well as Linux

![IBM Power E1080](https://mma.prnewswire.com/media/1608358/IBM_Power_E1080_Top_Down.jpg "E1080 Top Down View")

IBM typically has a family of servers, from small to large, with each processor generation, so expect IBM to follow up over the next year with 1-, 2-, and 4-socket server announcements.

This new E1080 server is big iron, ideal for the massive performance, memory, and throughput of big databases.  It's almost ideal for SAP HANA, but it will also replace older hardware being used for high transaction rate databases in the financial and healthcare markets.

IBM included two benchmark results as part of the announcement to prove the E1080's horsepower bona fides. One was a [CPU-oriented SPEC CPU2017 benchmark](http://spec.org/cpu2017/results/res2021q3/cpu2017-20210814-28679.html), the other an [SAP application benchmark](https://www.sap.com/dmc/benchmark/2021/Cert21059.pdf).  Both are widely-used industry-standard benchmarks, which gives lots of credences to IBM's best-in-class performance claims.

IBM also touted the system's security and hybrid cloud benefits.  The transparent memory encryption feature shows IBM's mettle when it comes to hardware-based security.  The hybrid-cloud benefit claim might not mean what people think, though, because Power10-based systems aren't available in any public clouds.  (The press release is ambiguous about whether IBM Power Virtual Server will eventually be based on Power10 systems.)

You'd expect IBM to announce full support for Red Hat's suite of software, including RHEL and OpenShift...which they did. But to me, the big reveal was a particuarly special integration with OpenShift.  The unique aspect of this integration isn't new technical features with OpenShift running on Power Systems, but rather a new option for licensing RHEL and OpenShift itself.

IBM has long offered a kind of consumption-based pricing for IBM Power Systems. Effectively, cores and memory can be disabled on some Power Systems servers, and you no longer have to pay for that portion of the hardware, nor for the operating system running on those cores.  (IBM has a number of different programs and ways to do this.)

WHat IBM and Red Hat jointly announced today was that they were extending this "on demand pricing" for RHEL and OpenShift. OpenShift itself can thus be consumed at on-demand pricing for systems running on-premises.  To be fair, Red Hat [hinted earlier this year](https://www.youtube.com/watch?v=KHUXlxhlWBM&t=2223s) that on-demand pricing would eventually be an option for on-premises OpenShift deployments, but this IBM E1080 announcement is the first public release of this licensing scheme.  And at the moment, this is an option available exclusively with the IBM hardware.

It's a great integration of best-in-class Red Hat PaaS software with best-in-class IBM server hardware. It proves there's traction in making IBM and Red Hat products work better together.  In fact, IBM might want ot take this one step further, and simply make OpenShift available at no cost on IBM Power hardware.
