---
layout: post
title:  "Detecting Drifts"
date:   2016-07-06 22:43:59
author: Eing Ong
categories: Testing
tags: tools
---
This blog describes how you can create an extension to [Hystrix Network Auditor Agent](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-network-auditor-agent) that may help you to be more productive in detecting dependencies not wrapped in Hystrix and prevent any drifts into failure over time. 

<h3>Why build an extension?</h3>
As teams started to adopt the Hystrix network auditor agent, I noticed the vast amount of time scrum teams take to incorporate it. The barriers or friction in productivity came from

* Figuring out where to add the event listener (especially for legacy code base) 
* Dependencies stack trace are lumped together with application logs i.e. parsing is needed
* Duplicate stack traces containing the same dependencies are logged i.e. 'sort \| uniq' commands are needed
* Stack trace is far too verbose as you don't really need the entire stack trace

Hence, the goal of the agent extension should fulfill the following objectives -

1. No application source code change required
2. Automatic self registration of listener
3. Duplicate dependencies are removed
4. Provides logging into a separate file
5. Dependencies can be configured (e.g. class name and lines count)
6. Logging format can be changed

The following is a step-by-step guide that I hope will be useful to those who have the same painpoints and want to achieve the same objectives. This should help you implement your own extension and customized to your needs.

<h3>Create HystrixNetworkAuditorAgent extension</h3>
I'm calling this extension "DriftDetector". It'll be the Java Agent that will be appended to your JVM command-line (instead of HystrixNetworkAuditorAgent). The important part of this is that you're creating a self registering event listener with "registerEventListener" method.

```
public class DriftDetector extends HystrixNetworkAuditorAgent {
    ...
    public static void premain(String agentArgs, Instrumentation inst) {
        HystrixNetworkAuditorAgent.premain(agentArgs, inst);
        if ((isAuditorLogEnabled == null) || !isAuditorLogEnabled.toLowerCase(Locale.getDefault()).contains("false")) {
            HystrixNetworkAuditorAgent.registerEventListener(driftEventListener);
        }
    }
```

<h3>Implement your event listener</h3>
This is where the bulk of the code is. The most important method is handleNetworkEvent() method. This method is called each time a network event happens and the event is not wrapped in Hystrix.

```
public class DriftEventListener implements HystrixNetworkAuditorEventListener {
    ...
    @Override
    public void handleNetworkEvent() {
        if (Hystrix.getCurrentThreadExecutingCommand() == null) {
            StackTraceElement[] stack = Thread.currentThread().getStackTrace();

            // Filter out the stack trace to specific classes you are interested in
            // Obtain unique dependencies from the filtered stack trace 
            // Write stack trace to log
            logStackTrace(filterStackTraceArray(stack));
        }
    }
```

<h3>Create list of useful properties to configure listener in command-line</h3>
Drift detector has to be used widely across different organizations, so it should be flexible enough to support all scrum teams without any code modification. By using Java system properties (by specifying with -D in JVM command line), each scrum team can customize these optional parameters according to their needs.

```
public enum PropertyConstant {

        // Dependencies to be identified. Default is "com.yourcompany".
        STACKTRACE_CLASSNAME ("drift.class", "com.yourcompany"),

        // Number of lines to print from the bottom of the stack trace.
        DRIFT_LINES_COUNT("drift.lines", "-1"),

        // By default, all stack trace logged are unique.
        REMOVE_DUPLICATES("drift.remove.duplicates", "true"),

        // By default, dependencies are logged.
        AUDITOR_LOG_ENABLED("drift.log.enabled", "true");

        private final String propertyName;
        private final String defaultValue;

        PropertyConstant(String propertyName, String defaultValue) {
            this.propertyName = propertyName;
            this.defaultValue = defaultValue;
        }

        public String getPropertyName() {
            return this.propertyName;
        }

        public String getDefaultValue() {
            return this.defaultValue;
        }
}
```

<h3>Create separate log file</h3>
To create a separate log file, you can add this snippet into your logback.xml for your project. Here's a sample snippet from logback.xml, and use *-Ddrift.logdir="..."* and *-Ddrift.file="..."* to specify the log file location.

```
    <appender name="DEPENDENCIES-FILE" class="ch.qos.logback.core.FileAppender">
        <file>${drift.logdir}/${drift.file}</file>
        <append>false</append>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg %n</pattern>
        </encoder>
    </appender>
```

In a few cases, we have large monolithic code base with a single jar and you'd rather not modify the log file configuration, here's how you can initialize the logger using JoranConfigurator -

```
        LoggerContext context = (LoggerContext) LoggerFactory.getILoggerFactory();
        JoranConfigurator jc = new JoranConfigurator();
        jc.setContext(context);
        context.reset();
        // Override default configuration and inject logfile name
        context.putProperty("drift.logfile", logFilename);
        context.putProperty("drift.logdir, logDir);
        try (InputStream resourceStream =
                    DriftEventListener.class.getResourceAsStream("dependencies.log")) {
            jc.doConfigure(resourceStream);
            logger = context.getLogger("DEPENDENCIES-FILE");
            resourceStream.close();
        } catch (JoranException je) {
            je.printStackTrace();
        } catch (IOException ioe) {
            ioe.printStackTrace();
        }
```

<h3>Create unit tests</h3>
Writing unit tests was a tad challenging as I'm extending from a class with private and static methods. I used JMockit to achieve over 80% code coverage and has so far not a single defect. Here's one JMockit test method -

```
    @Test
    public void getDependencies_setRemoveDuplicatesTrue_shouldRemoveDuplicates() {
        // Arrange: setup expectations and results to return
        Deencapsulation.setField(driftEventListener, "removeDuplicates", true);

        // Act: call method under test
        String actualResult = driftEventListener.getDependencies(stackTraceArrayWithDups);

        // Assert: expected string should not contain dups
        Assert.assertEquals(actualResult, expectedStringBufferWithoutDups.toString());
    }
```

<h3>Create jar package with dependencies</h3>
To build the Java Agent, you'll need to use gradle or maven to package all the dependencies needed during bootstrap classloading. Here's an example using maven shade plugin. One thing to note is that the jar file produced should be the same file name as specified in the MANIFEST.MF file, otherwise, you'll get NoClassDefFoundError for HystrixNetworkAuditorAgent.

```
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <shadedArtifactAttached>true</shadedArtifactAttached>
                    <shadedClassifierName>jar-with-dependencies</shadedClassifierName>
                    <artifactSet>
                        <includes>
                            <include>com.netflix.hystrix:*</include>
                        </includes>
                    </artifactSet>
                    <!-- Filename produced will be driftdetector.jar to be consistent with
                         Boot-Class-Path in MANIFEST.MF file -->
                    <finalName>${project.artifactId}</finalName>
                </configuration>
            </plugin>
```

<h3>Run it!</h3>
You should be able to run the agent by appending "-javaagent:${path}/driftdetector.jar" to JVM command-line. You'll have to exercise the application code (by running tests) so that dependencies will be captured.

To make DriftDetector run as part of your CI pipeline, you'll need to update server startup scripts (e.g. tomcat.start) to include the additional JVM configuration as well as deployment scripts (e.g. Chef) to download the drift detector jar to your build environment.

Please leave any feedback and comments. Hope this helps!
