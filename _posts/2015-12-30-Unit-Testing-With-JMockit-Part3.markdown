---
layout: post
title:  "In the deep trenches with JMockit Part 3"
date:   2015-12-30 19:43:59
author: Eing Ong
categories: Testing
tags: jmockit
---
This is a continuation from Part 2 on JMockit concepts. In this section, we'll cover the final important topics as well as references you can continue with.

<h2>Mockups</h2>
Mockups can be used when you have complex behavior of a class under test and your expectations start to get too convoluted. Note also that this is commonly referred to as a state-based approach.

~~~~
/**
 * Create a class containing all MockUps for UserAccount class.
 */
public class UserAccountMockHelper {
  /**
   * Create a method to allow user to specify the behavior for the class to be mocked.
   * In this case, we're faking Preferences class by faking its implementation.
   */
  public void setupUserAccountMockPreferences(final String country) {
    new MockUp<Preferences>() {
        // Creating the constructor's fake implementation
        @Mock
        public void $init() {}

        // Creating the static initializer blocks fake implementation
        @Mock
        public void $clinit() {}

        // Creating how getPreferences should behave in order to test.
        @Mock
        public Preferences getPreferences(String attribute) { 
             ...
        }

    // other MockUp Helpers for UserAccount class …
~~~~


<h2>The Non-accessibles</h2>
JMockit provides the capability to set private fields and invoke private methods. This is particularly useful if you work with legacy code base or code that is hard to be refactored.

| Source Code | JMockit test code | Description |
| ----------- | ----------------- | ----------- |
| private int intField; | setField(anInstance, "intField", 123); | Set value of a private field |
| private void static staticMethod() {} | invoke(SampleClass.class,"staticMethod"); | Invoke a private static method |
| private void instanceMethod(int val) {} | invoke(anInstance,"instanceMethod”, 987); | Invoke a private instance method |

<h2> References</h2>
This overview of JMockit is by no means exhaustive. You can read more in depth and particularly also "Partial Mocking" and "Using JMockit Code Coverage" in <a href="http://jmockit.org/" >JMockit</a> tutorial that is not covered here.
Here are some references that I started out with and still find them very useful today.
<ul>
<li><a href="http://jmockit.org/tutorial.html">JMockit official guide and tutorial </a></li>
<li><a href="http://jmockit.org/api1x/index.html">JMockit JavaDoc</a></li>
<li><a href="http://abhinandanmk.blogspot.com/2012/06/jmockit-tutoriallearn-it-today-with.html">Abhinandan's Blog</a></li>
<li><a href="http://dhruba.name/2009/11/08/jmockit-no-holds-barred-testing-with-instrumentation-over-mocking/">Dhruba Bandopadhyay's Blog</a></li>
<li><a href="https://blog.42.nl/articles/mockito-powermock-vs-jmockit/">Robert Bor's Blog on comparing Mockito/Powermock/JMockit with coding samples</a></li>
</ul>

