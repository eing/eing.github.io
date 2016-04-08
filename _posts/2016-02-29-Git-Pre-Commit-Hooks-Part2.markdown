---
layout: post
title:  "Git pre-commit hooks Part 2"
date:   2016-02-29 07:43:59
author: Eing Ong
categories: Technology
tags: git maven java jmockit
---
In Part 1, I use "mvn test" results to determine if pre-commit should pass. This leverages many maven plugins that are handy and easy to customize for your requirements. There are 3 checks I added - FindBugs, Checkstyle and JMockit coverage threshold. Let's get started with FindBugs first.

<h2>FindBugs plugin</h2>
Three key parameters of note are effort, threshold and failOnError. There's not much documentation on these fields at the [plugin website](http://gleclaire.github.io/findbugs-maven-plugin/examples/violationChecking.html) but you can find all configurations and default values [here](http://gleclaire.github.io/findbugs-maven-plugin/check-mojo.html). The configuration in maven plugin does not map to the original Ant settings. Gradle plugin though maps nicely to Ant settings. 

Anyways, I set effort to maximum for detecting all violations. Threshold has 3 settings - low, medium and high and I recommend low, otherwise why bother to use FindBugs? On failOnError, you can choose to set it to false so you can fix errors together with Checkstyle errors (and not sequentially) but be sure to add this check in Jenkins or your CI job.

You can refer to [Petri K's blog](http://www.petrikainulainen.net/programming/maven/findbugs-maven-plugin-tutorial/) with more details on this plugin.

If you're using gradle, take a look at [FindBugs Extension](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.quality.FindBugsExtension.html)

~~~text
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>findbugs-maven-plugin</artifactId>
                <version>3.0.3</version>
                <configuration>
                    <effort>Max</effort>
                    <threshold>Low</threshold>
                    <xmlOutput>true</xmlOutput>
                    <failOnError>true</failOnError>
                </configuration>
                <executions>
                    <!-- Ensures that FindBugs inspects source code when project is compiled. -->
                    <execution>
                        <id>analyze-compile</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
~~~

<h2>Checkstyle plugin</h2>
In Checkstyle, we created a customized rules list to be used across all projects. This xml file is committed as a git project and added as a dependency for the plugin which you can see below. Unlike FindBugs, [Checkstyle plugin](https://maven.apache.org/plugins/maven-checkstyle-plugin/examples/custom-developed-checkstyle.html) has very good documentation. All configurations and default values can be found [here](https://maven.apache.org/plugins/maven-checkstyle-plugin/check-mojo.html).

To customize your own rules set (if Sun rules set is too restrictive), refer to these [Checks](http://checkstyle.sourceforge.net/checks.html).

Again, for gradle, you can refer to [Checkstyle Extension](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.quality.CheckstyleExtension.html)

~~~text
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>2.17</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <linkXRef>false</linkXRef>
                    <configLocation>com/intuit/tools/checkstyle.xml</configLocation>
                    <consoleOutput>true</consoleOutput>
                    <violationSeverity>info</violationSeverity>
                </configuration>
                <executions>
                    <!-- Ensures that Checkstyle inspects source code when project is compiled. -->
                    <execution>
                        <id>checkstyle</id>
                        <goals><goal>check</goal></goals>
                        <phase>verify</phase>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>com.intuit.tools</groupId>
                        <artifactId>checkstyle-resources</artifactId>
                        <version>1.0.1</version>
                    </dependency>
                </dependencies>
            </plugin>
~~~

<h2>Surefire plugin for JMockit coverage threshold </h2>

Last but not least is code coverage threshold for unit tests. Since we are using JMockit for unit tests, we leverage JMockit-coverage that works very well with it. We have run into many issues using Jacoco and Cobertura plugins with JMockit.

JMockit has a good write up on code coverage, see [JMockit Code Coverage Tutorial](http://jmockit.org/tutorial/CodeCoverage.html). The only part that should be included is the surefire plugin and how you can set the coverage-check configuration. So, here're snippets extracted from my pom.

~~~text
          <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${surefire.version}</version>
                <configuration>
                    <systemPropertyVariables>
                        <coverage-metrics>all</coverage-metrics>
                        <!-- Threshold is set at 85% -->
                        <coverage-check>85</coverage-check>
                    </systemPropertyVariables>
                </configuration>
            </plugin>
          <plugin>
                <artifactId>maven-enforcer-plugin</artifactId>
                <executions>
                    <execution>
                        <id>coverage.check</id>
                        <goals><goal>enforce</goal></goals>
                        <phase>test</phase>
                        <configuration>
                            <rules>
                                <requireFilesDontExist>
                                    <files><file>coverage.check.failed</file></files>
                                </requireFilesDontExist>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
~~~

Other examples for coverage-check (instead of "85" in the example above) are 
<ul>
<li>"perFile:85,80,90", meaning that each source file must have at least 85% of line coverage, at least 70% branch coverage and at least 90% of data coverage </li>
<li>"package:85,80,90", has similar thresholds as above except that it is applied to source files in a given package, including sub-packages.</li>
</ul>

Finally, these are the dependencies I use along with the plugins.

~~~text
    <properties>
        <jmockit.version>1.21</jmockit.version>
        <surefire.version>2.19.1</surefire.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.jmockit</groupId>
            <artifactId>jmockit</artifactId>
            <version>${jmockit.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jmockit</groupId>
            <artifactId>jmockit-coverage</artifactId>
            <version>${jmockit.version}</version>
        </dependency>
    <dependencies>
~~~

<h2>Good sources of information</h2>
Yelp has a website dedicated just for pre-commit, [pre-commit.com](http://pre-commit.com). This goes to show how serious they are about good coding practices. 

For more indepth learnings on the concepts as well as other git hooks, they are well explained by [GitHooks](http://githooks.com) and [Atlassian](https://www.atlassian.com/git/tutorials/git-hooks/conceptual-overview).
