---
layout: post
title:  "In the deep trenches with JMockit Part 2"
date:   2015-12-15 18:43:59
author: Eing Ong
categories: Testing
tags: jmockit
---
This is a continuation from [Part 1](/testing/2015/11/18/Unit-Testing-With-JMockit-Part1/) on JMockit concepts. In this section, we'll cover Verifications, @Tested & @Injectable.

<h2>Verifications</h2>

Verifications comes after code under test and you are verifying against invocations that actually occurred during the test. Here is an example and it also shows yet another way to test the same use case as in Part 1 but now with NonStrictExpectations and Verifications.

~~~
    @Test
    public void login_3passwordMismatches_shouldRevokeAccount()
        throws Exception
    {
        new NonStrictExpectations() {
            {
                account.passwordMatches(anyString); result = anyBoolean;
            }
        };
        for (int i = 0; i < 3; i++) {
            service.login("john", "password");
        }
        new Verifications() {
            {
                account.setRevoked(true);
            }
        };
    }
~~~

There are four different forms of Verifications. Here's a table to make it easy to read and compare the different variations. 

|     | Verifications | VerificationsInOrder | FullVerifications | FullVerificationsInOrder | 
| --- | ------------- | -------------------- | ----------------- | ------------------------ |
| Checks order | No | Yes | No | Yes |
| Checks invocation count | 1 or more unless "times=x" is added | Similar as Verifications | Exact invocations | Similar as FullVerifications|
| Verify specified mocked types | No | No | Yes | Yes |

On "Verifying specified mocked types", FullVerifications can take in the mocked type and verify the invocations that occur for that particular mocked field and not the mocks that you don't care about.

~~~
  new NonStrictExpectations() {
      {
         ...
      }
  };

  // Verify invocations on mocked account.
  // You can also use it to verify that no invocations occur on this 
  // account if there is no code in this block.
  new FullVerifications(UserAccount account) {
    {
    }
  };
~~~

<h2>@Tested and @Injectable</h2>
Now, we introduce the next two annotations - Tested & Injectable. 

~~~
public class LoginServiceTest {
    // Class under test: LoginService
    @Tested LoginService service;
    ...
~~~

What does @Tested annotation do for LoginService?

- LoginService is automatically instantiated
- Injectable dependencies of LoginService are injected after LoginService is instanstiated

Tip: Classes declared final are <b>non-injectable</b>.

Here's a code sample to explain the these two annotations along with @Mocked

~~~
public class LoginServiceTest {
    // Class under test: LoginService
    @Tested LoginService service;
 
    // Dependency injected in code under test
    @Injectable AuthService authService;

    // Dependency instantiated in code under test
    @Mocked UserAccount account;
~~~

<h4>@Tested(fullyInitialized=true)</h4>
In some cases where you did not specify Mocked or Injectable for every field of the tested object but would want JMockit to assigned a <b>non-null</b> value, you can use the attribute "fullyInitialized".  JMockit will create and recursively initialize a suitable real instance.

<h4>Injectable versus Mocked</h4>
This is a common question - what are the differences between Injectable and Mocked? The table below outline the differences -

|     | @Injectable <br>AuthService authservice| @Mocked<br>AuthService authservice |
| --- | --------------------- | ------------ | ------------------ |
| Constructors | Not mocked | Mocked |
| Static fields and methods | Not mocked | Mocked |
| Instance methods | Mocked | Mocked |
| Instances | Only 1 AuthService instance is mocked in scope | All instances of AuthService type are mocked in scope |


<h2>@Mocked revisited</h2>
This is also another common question. Should I mock a field in class or method level?

| public class LoginTest {<br>&nbsp;&nbsp;&nbsp;&nbsp;@Mocked UserAccount account;| @Test<br>public void testLogin(@Mocked final UserAccount account)  |
| --------------------- | ------------ |
| Mock field in test <b>class</b> | Mock parameter in test <b>method</b> |
| All UserAccount instances in class are mocked | UserAccount mock instance has method level scope |
| account is automatically created by JMockit and assigned | account is automatically created by JMockit and passed by JUnit/TestNG test runner when calling test method|
| account can be null | account is <b>never</b> null| 
| account does <b>not</b> support cascaded static factory methods, e.g. UserAccount.getInstance();| account supports cascaded static factory methods |

<h4>@Mocked(stubOutClassInitialization=[true|false])</h4>
Part 1 has highlighted that "stubOutClassInitialization" is false by default. Here's an example to demonstrate static fields and static initializer -

~~~
    public class UserAccount {
        // Any static fields regardless of access modifiers are by default initialized
        public static final String DEFAULT_REGION = REGION.NORTH_AMERICA.name();
        protected static String country;
        private static String state;
        
        // Any static classes are by default initialized
        static  {
            country = Config.getString("country");
        }
    }

~~~

Here are the differences in the tests -

~~~
    /** 
     * Default @Mocked annotation
     *   i.e. @Mocked(stubOutClassInitialization=false) UserAccount account
     */
    @Test
    public void login_3passwordMismatches_shouldRevokeAccount(
        @Mocked UserAccount account) throws Exception
    {
        // Fields - DEFAULT_REGION, country and state are initialized by default.
        // Static initializer (calling Config.init()) is initialized by default.
        assertThat(UserAccount.DEFAULT_REGION, is("NORTH_AMERICA"));
        assertThat(UserAccount.country, is("US"));
    }

    /** 
     * Adding @Mocked(stubOutClassInitialization=true) to UserAccount account.
     * This is used when you need to create the behavior and expectation for 
     * these fields for testing for example. 
     */
    @Test
    public void login_3passwordMismatches_shouldRevokeAccount(
        @Mocked(stubOutClassInitialization=true) UserAccount account) throws Exception
    {
        // Fields - DEFAULT_REGION, country and state are null. 
        // Static initializer (calling Config.init()) is null.
        Assert.assertNull(UserAccount.DEFAULT_REGION);
        Assert.assertNull(UserAccount.country);
    }
~~~

This concludes JMockit Part 2. In the final part of [In the deep trenches with JMockit Part 3](http://eing.github.io/testing/2015/12/30/Unit-Testing-With-JMockit-Part3/), we'll look at MockUp, non-accessibles and useful references and blogs on JMockit.

Here are the links to all three deep trenches on JMockit  -

* [In the deep trenches with JMockit Part 1 - On Mocked and Expectations](/testing/2015/11/18/Unit-Testing-With-JMockit-Part1/)
* [In the deep trenches with JMockit Part 2 - On Verifications, @Tested, @Injectable and more on @Mocked in depth](/testing/2015/12/15/Unit-Testing-With-JMockit-Part2/)
* [In the deep trenches with JMockit Part 3 - On Mockups, non-accessibles and references](/testing/2015/12/30/Unit-Testing-With-JMockit-Part3/)
