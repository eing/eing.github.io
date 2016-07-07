---
layout: post
title:  "Detecting Drifts"
date:   2016-07-06 22:43:59
author: Eing Ong
categories: testing
tags: tools
---
This blog describes how you can create an extension to [Hystrix Network Auditor Agent](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-network-auditor-agent) that may help you to be more productive in detecting dependencies not wrapped in Hystrix and hence, drift into failure over time. 

<h3>Why build an extension?</h3>
As teams started to adopt the Hystrix network auditor agent, I noticed the vast amount of time scrum teams take to incorporate it in. The friction in productivity came from

* Figuring out where to add the event listener (especially for a legacy code base) 
* Dependencies stack trace are lumped together with application logs i.e. parsing is needed
* Duplicate stack traces containing the same dependencies are logged i.e. 'sort \| uniq' commands are needed
* Stack trace is far too verbose as you don't really need the entire stack trace

Hence, the goal of the agent extension should fulfill the following objectives -

1. No application source code change required
2. Automatic self registration
3. Duplicate dependencies are removed
4. Provides logging into a separate file
5. Dependencies can be configured (e.g. class name and lines count)
6. Logging format can be changed

The following is a step-by-step guide that I hope will be useful to those who have the same painpoints and want to achieve the same objectives. This should help you implement your own extension customized to your needs.

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
            ArrayList<String> filteredStack = filterStackTraceArray(stack);

            // Obtain unique dependencies from the filtered stack trace 
            String dependencies = getDependencies(filteredStack);

            // Write stack trace to log
            if (dependencies.length() > 1)
                logger.info(dependencies);
        }
    }
```

<h3>Create list of useful properties to configure listener during runtime</h3>
Drift detector has to be used by every scrum team in the company, so it should be flexible enough to support all scrum teams without any code modification. By using Java system properties (by specifying with -D in JVM command line), each scrum team can customize these optional parameters according to their needs.

```
    // Property to configure log filename
    public static final String PROPERTY_LOGFILE_NAME = "drift.file";
    // Default name of log file i.e. dependencies.log
    public static final String DEFAULT_LOGFILE_NAME = "dependencies.log";

    // Log directory to place dependencies.log
    public static final String PROPERTY_LOG_DIR_PATH = "drift.logdir";

    // Dependencies to be identified. Default is "com.yourcompany".
    public static final String PROPERTY_STACKTRACE_CLASSNAME = "drift.class";
    public static final String DEFAULT_STACKTRACE_CLASSNAME = "com.yourcompany";

    // Number of lines to print from the bottom of the stack trace. Default is -1 i.e. all lines.
    public static final String PROPERTY_DRIFT_LINES_COUNT = "drift.lines";
    public static final int DEFAULT_DRIFT_LINES_COUNT = -1;

    // By default, all stack trace logged are unique. To disable (i.e. do not check for duplicates), set this flag to false.
    public static final String PROPERTY_REMOVE_DUPLICATES = "drift.remove.duplicates";

    // By default, dependencies are logged. To disable, set this flag to false.
    public static final String PROPERTY_AUDITOR_LOG_ENABLED = "drift.log.enabled";
```

<h3>Create log file</h3>
Here's a snippet from logback.xml. The idea is that the logger context should have properties "drift.logdir" and "drift.file" in order to create this log file.

```
    <appender name="DEPENDENCIES-FILE" class="ch.qos.logback.core.FileAppender">
        <file>${drift.logdir}/${drift.file}</file>
        <append>false</append>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg %n</pattern>
        </encoder>
    </appender>
```

Here's how the listener initialized the logger using JoranConfigurator -

```
        LoggerContext context = (LoggerContext) LoggerFactory.getILoggerFactory();
        JoranConfigurator jc = new JoranConfigurator();
        jc.setContext(context);
        context.reset();
        // override default configuration and inject logfile name
        context.putProperty(Constant.PROPERTY_LOGFILE_NAME, logFilename);
        context.putProperty(Constant.PROPERTY_LOG_DIR_PATH, logDir);
        try (InputStream resourceStream =
                    DriftEventListener.class.getResourceAsStream(Constant.DEFAULT_LOGBACK_CONFIG_FILENAME)) {
            jc.doConfigure(resourceStream);
            logger = context.getLogger(Constant.LOGGER_NAME);
            resourceStream.close();
        } catch (JoranException je) {
            je.printStackTrace();
        } catch (IOException ioe) {
            ioe.printStackTrace();
        }
```


<h3>Create unit tests</h3>
Writing unit tests was a tad challenging as I'm extending from a class with private and static methods. I used JMockit to achieve over 80% code coverage and has so far no single defect in the source code. Here's one JMockit test method -

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
                    <artifactSet>
                        <includes>
                            <include>ch.qos.logback:*</include>
                            <include>io.reactivex:*</include>
                            <include>com.netflix.hystrix:*</include>
                            <include>com.netflix.archaius:archaius-core</include>
                            <include>com.netflix.rxjava:*</include>
                            <include>org.slf4j:slf4j-api</include>
                            <include>commons-logging:*</include>
                            <include>commons-lang:*</include>
                            <include>commons-configuration:*</include>
                        </includes>
                    </artifactSet>
                    <!-- Filename produced will be driftdetector.jar to be consistent with
                         Boot-Class-Path in MANIFEST.MF file -->
                    <finalName>${project.artifactId}</finalName>
                </configuration>
            </plugin>
```

<h3>Run it!</h3>
You should be able to run the agent by appending " -javaagent:${path}/driftdetector.jar" to JVM command-line. You'll have to exercise the application code (by running tests) so that dependencies will be captured.

Hope this helps!
