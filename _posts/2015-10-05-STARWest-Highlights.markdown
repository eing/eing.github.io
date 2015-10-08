---
layout: post
title:  "STARWEST 2015"
date:   2015-10-06 07:43:59
author: Eing Ong
categories: Conferences
tags: conferences starwest
cover:  "assets/starwest.png"
---
<h1> Highlights from <a href="http://starwest.techwell.com" >STARWEST</a> 2015</h1>

On the whole, I learnt more in this conference than I did 3-4 years ago. You can easily tell when I tweet more and blog more. Here are some of the more prominent messages I gathered from the conference.

<h2> Definition of testing (that is different from automation)</h2>

From <i>"The Testopsy: Dissect Your Testing"</i> by James Bach and Jon Bach,

`Testing is never to get from Point A to Point B but to discover what was previously unknown and to notice them when they do through experimentation and exploration including questioning, modeling, observation, inference etc.`

They suggested an approach to examine if your testing is effective with a <b>Testopsy</b>. This is a creative twist of an autopsy (as you can expect from James & Jon Bach), where a testopsy is an autopsy of a testing session. Here's how it goes,
  
<ol>
<li>Record a session of your testing.</li>
<li>Note down every single activity that you do. Put specific words to each activity.</li>
<li>Replay the video and critic your own testing process.</li>
</ol>

This helps us dissect our testing into conscious and unconscious testing. Whether you are testing one factor at a time (o-fact) or many factors at a time (m-fact), boundary testing, and how much time are you actually testing or spending b-time (bug investigation time) or s-time (setup time) where both activities impact productivity in testing.

More on Testopsy from the <a href="http://starwest.techwell.com/sessions/starwest-2015/testopsy-dissect-your-testing">session abstract</a>,

`Using a Testopsy, you build your skills of observation, narration, and test framing. And if you do it with a colleague or as a group, it stimulates discussion on test design and test strategy. Much like a medical examiner narrates his autopsies into a tape recorder, you can look very carefully at what you actually do and identify your own heuristics. By putting that process into descriptive, evocative words, you can discover surprising depths in each act of testing you perform.`

<h4>Deeper dive into the definition of testing</h4>

From Michael Bolton and James Bach session on "The Secret Life of Testers: Where Your Time Really Goes", testing terminology is lumped into many activities that is not testing. Hence, this session shows how testing can be "de-lumped" so that we can be cognizant of the time that is taken away from actual testing for better test effort estimation.

Examples of time taken away from actual testing - bugs, test setup, attempt to perform testing such as discussing test plans, code changes, and attending related chalk talks and fixing test scripts.

They demonstrated a tool to visually animate the progress of testing by charting the testing sessions, here's how it goes -

<ol>
<li>Break down a week into "deci"-session blocks.</li>
<li>Define different color codes for actual testing, bugs, setup, update, re-test and re-setup.</li>
<li>Mark each block according to the activity you are doing.</li>
</ol>

This will help better predict testing time, predict all the builds, changes, bugs and provide room for tacit, implicit and discovered work and not just the explicit work or plan. This will also help to set better expectations for the team. 

<h2>Bringing or Adding Value to the Product</h2>
Janet Gregory's "I Don’t Want to Talk about Bugs: Let’s Change the Conversation" was the best keynote in this conference. She wants the change the conversation (from defects) to bringing more visibility to the value testing brings to the product. Here are some notable quotes from Janet's talk,

`Counting defects tells us how bad our product quality is, but it doesn't tell us how good it is.`

`Bring visibility to how testing add value to our business by reducing risks and misunderstandings.`

`Are you looking ahead for risks or are you following behind looking for bugs?`

`As testers to add value to our business, we should ask: can something be right (meet product specs) but not meet customers' needs. (From @walkeTim: Does the 'inspected by' slip in a pair of trousers mean the trousers fit or look good on you?)`


Janet listed a few ways one can access product value and if our testing has added value -
<ul>
<li>Customer satisfaction</li>
<li>Customer retention</li>
<li>The type of issues found in production</li>
<li>User analytics on who uses your product and do they use it in the way you expected</li>
</ul>

<h2>Service Virtualization</h2>
There are a few talks on service virtualizations (e.g. IBM, Parasoft). Service virtualization essentially is stubbing out a service (e.g. recording the response to be played back) so that the service/component/application under test do not have to rely on this service if it is not completed, still evolving, controlled by a third party, or has restricted/costly/non-dedicated access.

Co-incidentally, two of my co-workers are presenting at <a href="https://www.portal.reinvent.awsevents.com/connect/sessionDetail.ww?SESSION_ID=1727">AWS re:Invent</a> on this using an open source tool, <a href="http://wiremock.org">WireMock</a>.

<h2>Automation</h2>
Noticed that I left out the word "Test" in front of Automation and that I prepared you in the first heading. Here are some notable quotes from sessions listed above as well as the keynotes.

From Bj Rollison's keynote,
`Automation is valuable when you have large amounts of data and data variability, stress, performance, concurrency.`

`Automation is a beast and needs to be controlled before it gets out of hand (by dividing into appropriate test suites).`

From Michael Bolton/James Bach session,
`Machines executes checks but human tests.`

From Bart Knaack's keynote, he strikes out 'Test' in 'Test Automation' and replaces it with 'Check',
`Test Automation ==> Check Automation`

These clearly dismiss the notions of the automation magic wand that will replace all testing efforts or that testing is dead because there's automation.

<h2>REST Services (check) Automation</h2>
I have to put a plug in for this since I presented <a href="http://starwest.techwell.com/sessions/starwest-2015/automate-rest-services-testing-restassured"> "Automate REST Services Testing with RestAssured"</a>. The open source tool I talked about was <a href="https://github.com/jayway/rest-assured">REST-assured</a>. I'll have more on this topic once I migrated all my blogs from eingong.blogspot.com.

<ul>
<li>Slides are available at <a href="http://www.slideshare.net/eingong/2015starwest-presentation-on-restassured">SlideShare</a></li>
<li>RESTAssured CLI tool to help you generate project structure, build dependencies and sample tests is at <a href="https://github.com/eing/restassured_cli">github.com/eing</a></li>
</ul>
