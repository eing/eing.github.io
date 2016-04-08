---
layout: post
title:  "Metrics to drive change in behavior - Part 1"
date:   2016-03-29 07:43:59
author: Eing Ong
categories: Metrics
tags: Sonar Perforce TechDebt
---
I'll talk about just two metrics in this two part blog. I had to obtain different metrics for a Quality 6 Pager I'm working on, a narrative on our current state of quality, and our target state in the organization. This concept came from [Amazon](https://www.linkedin.com/pulse/beauty-amazons-6-pager-brad-porter) and you can easily find more information on it.

The first is technical debt of a monolith. Many of the Sonar reports we have are based on individual component and it makes sense as such reports take time to generate. However, a Sonar report on the entire monolith has not been done and I had to go through some hoops to get the data.

<h2>Finding technical debt for monoliths</h2>

To install and run Sonar on your local is straightforward. Download from [SonarQube](http://www.sonarqube.org/downloads/), and run the command and then go to http://localhost:9000.

~~~text
nohup ${path}/sonarqube-5.4/bin/macosx-universal-64/sonar.sh console&
~~~

The first error I got was from Sonar's attempt to contact my SCM host

~~~text
SCM provider was set to "svn" but no SCM provider found for this key. No SCM provider installed.
~~~

You have to login to Sonar's admin and disable this feature ("General Settings > SCM") or setting property sonar.scm.disabled=true. See more from [Sonar Support](http://docs.sonarqube.org/display/SONARQUBE50/SCM+support) on this issue.

The second step is to setup up your project to run with Sonar Scanner. You have to create a "sonar-project.properties" file in the project root that you want to scan. You can see that you can add inclusons & exclusions.

~~~text
sonar.projectKey=yourProject:yourProjectComponenet
sonar.projectName=yourProject
sonar.inclusions=**/*.java
sonar.exclusions=**/target/**,**/*.js
~~~

The final step is to run "sonar-runner". For me, I like minimal typing so I added the following in my .bashrc

~~~text
In .bashrc:
export SONAR_RUNNER_HOME=$HOME/Downloads/sonar-scanner-2.5
export PATH=$PATH:$SONAR_RUNNER_HOME/bin

In project root, run:
sonar-runner
~~~

As expected, this failed after some time with out of memory error.

~~~text
Caused by: java.util.concurrent.ExecutionException: java.lang.OutOfMemoryError: Java heap space
~~~

After some research, I added this into sonar-project.properties, and managed to obtain the metrics I was looking for - technical debt (number of days to fix) & number of critical issues, see [sample here](http://docs.sonarqube.org/display/SONAR/Technical+Debt) from Sonar.

~~~text
export SONAR_RUNNER_OPTS="-Xmx1G -Xms1024m -XX:MaxPermSize=1024m -XX:-UseGCOverheadLimit"
~~~

In the second part of this metrics blog, I'll be looking for data to better understand engineers behavior.
