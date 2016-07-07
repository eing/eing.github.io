---
layout: post
title:  "Testing for Resiliency"
date:   2016-06-10 19:43:59
author: Eing Ong
categories: Testing
tags: tools
---
<h2>Understanding Resiliency</h2>
Many tests stop at unit and business use cases workflow with both positive and negative scenarios. In the world of microservices where a webservice depends on many other microservices to accomplish a complex task, building a resilient service is an implicit customer expectation that we need to build in, just like any feature.

First, let's take a look at a few definitions for resiliency.

[Resilience](https://en.wikipedia.org/wiki/Resilience_(network)) (from wikipedia)
 : > Resilience is the ability to provide and maintain an acceptable level of service in the face of faults and challenges to normal operation. 

A great book to read on resiliency is [Release It!](https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213) by Michael T. Nygard where he wrote - 

: > A resilient system keeps processing transactions, even when there are transient impulses, persistent stresses, or component failures disrupting normal processing. This is what most people mean when they just say stability. It’s not just that your individual servers or applications stay up and running but rather that the user can still get work done.

Understanding resiliency is the first step, what comes next? Here are some questions that I would ask -

 * How do I find out the services that threaten my service stability?
 * How do I simulate instability of the services that I depend on?
 * How can I ensure that the stability of my service does not regress/drift over time?

There are many tools that tackle each question (note. there is not one tool that does all three well). In the next section, I'll list the tools that you might be interested to look into.

<h3>Discovery of dependencies</h3>
Finding out dependencies should be the first and foremost step for resiliency testing. Here is the list of popular tools -

 * [Charles](http://www.charlesproxy.com)
 * [Fiddler](http://www.telerik.com/fiddler)
 * [tcpdump](http://www.tcpdump.org)
 * [Andiparos/Zap Proxy](https://code.google.com/p/andiparos/, https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project)
 * [Hystrix Network Auditor Agent](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-network-auditor-agent)

Hystrix Network Auditor Agent was the tool we went with as our application is built with Hystrix fault tolerance library. I will describe in my next blog an extension I wrote to the audtior agent so that you do not have to modify application code as well as other productivity improvements.

<h3>Simulating instability conditions</h3>

Depending on the application/service you are testing, you may need a proxy for http, and/or non-http. If your services are hosted in AWS, [Chaos Monkey](https://github.com/Netflix/SimianArmy/wiki/Chaos-Monkey) from Netflix is hands-down the most aggressive resiliency testing tool.

<h4>HTTP proxies</h4>
For HTTP dependencies, here are some criteria to consider when evaluating the tools

 * Supports whitelist, ignore, exclude hosts
 * Supports http and https traffic proxy
 * Forward proxy traffic through another proxy
 * Apply latency delay by target host and port
 * Apply latency delay by http operation
 * Record request and response for stubbing
 * Configurable delay (short/long/before timeout/after timeout etc.)
 * Customizable response (body, multipart, header, response code etc.)
 * Supports matching rules for hostname, port, uri (full/partial), body, multiplicity
 * Easy to automate
 * No modification of application code
 * Supports various fault injection
 * Detailed logging of traffic and payload

Some popular open source tools are

 * [WireMock](http://wiremock.org)
 * [MockServer](http://www.mock-server.com)
 * [Node http proxy](https://github.com/nodejitsu/node-http-proxy)
 * [BetaMax](http://freeside.co/betamax/)
 * [MITM](https://mitmproxy.org)

WireMock and Node http proxy are both promising tools. Note that WireMock only supports one http dependency per WireMock instance.

<h4>Non-http proxies</h4>
For non-HTTP dependencies (e.g. UDP/TCP, SOCKS5), in addition to the criteria above, here are some additional considerations that may apply to you -

 * Reverse DNS lookup of IP addresses
 * Supports closing of sockets
 * Able to listen to websockets

Available open source tools are

 * [ToxiProxy](https://github.com/Shopify/toxiproxy)
 * [Netty](https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example/socksproxy)
 * [Go Any Proxy](https://github.com/ryanchapman/go-any-proxy)
 * [Saboteur](https://github.com/tomakehurst/saboteur)
 * [Destructive Socks Proxy](https://github.com/tawawhite/destructive_socks5_proxy)

<h3>Other resiliency testing tools</h3>
There are a few other categories worth noting and they may apply to your testing. They are

 * JDBC proxy
   - [SSL-SQL-Proxy-Server](http://leechuck.de/proxy/)
   - [P6Spy](http://p6spy.github.io/p6spy/2.0/install.html)
   - [HA-JDBC](http://ha-jdbc.github.io)
   - [Proxool](http://proxool.sourceforge.net)
   - [Virtual JDBC](http://vjdbc.sourceforge.net)
   - [LDBC](http://ldbc.sourceforge.net)
 * Platform dependent tools 
   - Mac e.g. [ipfw/pfctl](http://www.joemiller.me/2010/08/31/simulate-network-latency-packet-loss-and-bandwidth-on-mac-osx)
   - Linux e.g. [tc](http://linux.die.net/man/8/tc), iptables
 * Monitoring
   - [Hystrix Dashboard](https://github.com/Netflix/Hystrix/tree/master/hystrix-dashboard)
   - [Splunk Dashboard](http://www.splunk.com)
   - [Graphite + CodaHale/DropWizard Metrics](http://metrics.dropwizard.io/3.1.0/manual/graphite/)
   - [Glimpse](http://getglimpse.com)
   - [Zipkin](https://twitter.github.io/zipkin/index.html)
   - [MoSKito](http://www.moskito.org)
   - [Java Simon](https://code.google.com/p/javasimon/)
   - [Servo](https://github.com/Netflix/servo)

<h3>Conclusion</h3>
Once you have a good understanding of what you need to test for resiliency, I hope that by shortlisting these tools by category and criteria will help you narrow down and pick the right tool for your resiliency testing toolbox. Having a set of principles when evaluating tools is also highly recommended. For example, our principles are 

* Prioritize mature OST(Open Source Tools) over COTS (Commercial Off-The-Shelf) and COTS over homegrown DIY
* Test development using these tools should be simple, easy and the resulting code readable
* Tools should be easily extended, customized, and/or a thin abstraction layer added to serve application specific needs
* Tools must be a cost effective solution considering purchase, training, development and maintenance costs
* Tools should be scalable to enable and increase engineers productivity in all scrum teams

I welcome any questions, insights or suggestions you have. Please feel free to email me or leave comments on this blog. Thank you for reading!
