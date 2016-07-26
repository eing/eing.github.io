---
layout: post
title:  "In the deep trenches with JMockit Part 1"
date:   2015-11-18 17:43:59
author: Eing Ong
categories: Testing
tags: jmockit
---
JMockit is a class loader remapping toolkit that you can use to replace your dependency classes by remapping them to your desired behavior when unit testing. Under the hood, JMockit uses Java instrumentation API to make direct byte code modifications at runtime. 

JMockit framework abstracts this complex capability with its comprehsive APIs to allow engineers to mock interfaces, classes, enums, fields, methods, constructors, and accessors from public, private, protected, static, and final.

Finally, I'd like to point out that JMockit syntax is similar to a DSL (domain specific language) that users need to be familiar with.

I'll be blogging the following important concepts in an order easier to learn and help answer questions. Hope this will help you too.

<h2>@Mocked</h2>
~~~~java 
public class LoginServiceTest {
    @Mocked UserAccount account;
~~~~
@Mocked is the most commonly used annotation in JMockit. In the example above, UserAcount instance, "account" is annotated with Mocked. What does this really mean, i.e. What is mocked for UserAccount instance?
<ul>
<li>All UserAccount methods (whether it is private/final/abstract/protected/static and constructors) and all its <b>non-static</b> initializers and <b>non-static</b> fields assignment (e.g. constructors)</li>
<li>All of UserAccount super classes (if any) up to but not including java.lang.Object will be mocked recursively.</li>
</ul>
Tip: To mock [static initializers](http://www.developer.com/java/other/article.php/2238491/The-Essence-of-OOP-using-Java-Static-Initializer-Blocks.htm), you have to use @Mocked(stubOutClassInitialization=true). By default, it is false. See Part 2 for an example.

<h2>Expectations</h2>
When we are unit testing a class, we want it to run reliably, repeatedly and fast. Typically, our class under test will depend on other classes/packages/jars that might be unreliable, expensive to setup/run, or are slow to response. Hence, we need a means to specify the behavior of these external APIs and JMockit provides Expectations to do just that.

~~~
   @Mocked UserAccount account;
   @Test
   public void login_3passwordMismatches_shouldRevokeAccount() throws Exception
    {
        new Expectations() {
            {
                account.passwordMatches(anyString); result = false; minTimes = 3;
                account.setRevoked(true);
            }
        };
        for (int i = 0; i < 3; i++) {
            service.login("john", "incorrect_password");
        }
    }
~~~

<h4>More examples using Expectations</h4>

| Expectations examples | Return result(s) | What matches |
| --------------------- | ---------------- | ------------ |
| <b>Constructor</b><br>new UserAccount(anyInt);<br>minTimes = 1; maxTimes = 2; | void | new UserAccount(1); <br> new UserAccount(2); |
| <b>Method call</b><br>account.passwordMatches(anyString);<br>result = new Exception(); times = 1; | Exception instance | @Mock or @Injectable account |
| <b>Mocked instance</b><br>onInstance(account).passwordMatches(anyString);<br> returns(false, true); | 1st call = false.<br>2nd call = true. | @Mock or @Injectable account|


<h4>Different Types of Expectations</h4>

|     | NonStrictExpectations | Expectations | StrictExpectations |
| --- | --------------------- | ------------ | ------------------ |
| Checks order | No | Yes | Yes |
| Checks invocation count | No<br>0 or more invocations unless “times=x” is added | Yes<br>At least 1 invocation | Yes<br>Exactly 1 invocation unless “times=x” is specified | 
| Use with Verifications | Definitely | Recommended | Not necessary |


<h4>Expectation Argument Matches</h4>

| 10 "any" argument/result matchers | 16 "with" argument matchers |
| --------------------------------- | --------------------------- |
| any | withAny (T arg) |
| anyBoolean | withArgThat(Matcher argument Matcher |
| anyByte | withCapture(List<T> valueHolderForInvocations |
| anyChar | withEqual(double value, double delta) |
| anyDouble | withEqual(float value, float delta) |
| anyFloat | withEqual(T arg) |
| anyInt | withInstanceLike(T object) |
| anyLong | withInstanceOf(Class<T> argClass) |
| anyShort | withMatch(T regex) |
| any String | withNotEqual(T arg) |
| | withNotNull() |
| | withNull() |
| | withPrefix(T text) |
| | withSameInstance(T object) |
| | withSubstring(T text) |
| | withSuffix(T text) |
 
This concludes JMockit Part 1 and I'll cover more topics in the next 2 parts of this JMockit blog -

* [Part 2 - On Verifications, @Tested, @Injectable and more on @Mocked in depth](/testing/2015/12/15/Unit-Testing-With-JMockit-Part2/)
* [Part 3 - On Mockups, non-accessibles and references](/testing/2015/12/30/Unit-Testing-With-JMockit-Part3/)
