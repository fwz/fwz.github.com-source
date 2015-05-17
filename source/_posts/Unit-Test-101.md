title: "Unit Test 101"
date: 2015-05-09 16:08:20
tags: Unit Test
categories: 
- Engineering
- Engineering Excellence
---

![](http://wenzhong.qiniudn.com/img/blog/testing.png)

The importance of writing unit tests have been heavily discussed. However, reality is cruel. Many developers do not write UT, and even larger amount of developers write funny tests just to pass the code coverage bar, and some developers with ambition are not quite sure about how to write good unit tests. Most good UT are written by top developers.

I thought I am that kind of ambitious developer, so I spend two weekends to start to learn it. Most cases listed below are originated from or revised from the book ["Practical Unit Testing with Testng and Mockito"](http://www.amazon.com/Practical-Unit-Testing-TestNG-Mockito/dp/839348930X) by [Tomek Kaczanowski](http://kaczanowscy.pl/tomek/). A really good book worth reading.

---

## Concepts
Before we start, let's visit the concepts of SUT and DOC.

<!-- more -->

### SUT & DOC

SUT, or **System Under Test**, are understood as **the part of the system being tested**. Depending on the type of test, SUT may be of very different granularity – from a single class to a whole application. 

DOC, or **Depended On Component**, is any entity that is required by an SUT to fulfill its duties.

For example, recently I am working with [Spring](https://spring.io/), and I am implementing a Filter class, which take in a `ServletRequest` object, a `ServletResponse` object and a `FilterChain` object. Then when writing unit test, SUT is the `Filter` class I am working on. DOC are consist of `ServletRequest` and a `ServletResponse`, and a `FilterChain`.

### Test Type
There's many types of tests(and names) in software development, but the most important tests are: 

* **unit test** which help to ensure high-quality code
* **integration test** which verify that different modules are cooperating effectively
* **end to end test** which put the system through its paces in ways that reflect the standpoint of users

Simple. Let's start to discuss UT (Unit Test).

---

## Unit Test

### Structure
The structure of writing a UT is:
* **Arrange** (Setup environment / foundation of the test)
* **Act** (run the test)
* **Assert** (compare expect result with actual result)

### Frequency of running UT
UT are supposed to be run repeatedly and frequently. Even someone is not using TDD, after we have implemented any feature, all the UT under this project should be run.

### Bad smells

A test is not a unit test if:
* It talks to the database
* It communicates across the network
* It touches the file system
* It can’t run at the same time as any of your other unit tests
* It might fail even the code is correct, or it might pass even the code is not correct.

The above points make writing UT not quite straight forward. To make our UT clean, some knowledge such as Mocking is needed. Also we need to master are some skills to utilize the power of our test framework.

### Framework and tools
Base on some investigation, I choose TestNG and Mockito and finally spot this book. This is just personal preferences. JUnit provide mostly the similar function with TestNG. Something we desperately need should be available for a mature framework.

---

Let's start coding to demonstrate what we have listed above. Here is a `Money` class, with two properties, amount and currency.

### Compatability

{%codeblock lang:java Money.java %}

public class Money {
    private final int amount;
    private final String currency;

    public Money(int amount, String currency) {
        if (amount < 0) {
            throw new IllegalArgumentException("illegal negative amount: [" + amount + "]");
        }
        if (currency == null || currency.isEmpty()) {
            throw new IllegalArgumentException("illegal currency: [" + currency + "], it can not be null or empty");
        }

        this.amount = amount;
        this.currency = currency;
    }

    public int getAmount() {
        return amount;
    }

    public String getCurrency() {
        return currency;
    }

    public boolean equals(Object o) {
        if (o instanceof Money) {
            Money money = (Money) o;
            return money.getCurrency().equals(getCurrency()) && getAmount() == money.getAmount();
        }
        return false;
    }
}
{%endcodeblock%}

Now let's write a very simple test to test whether the constructor work as expect. Here our test are with TestNG and JUnit style (with support in TestNG).

{%codeblock lang:java MoneyTest.java%}
import org.testng.annotations.Test;
import static org.testng.Assert.assertEquals;

import org.testng.AssertJUnit;
@Test
public class MoneyTest {
    public void constructorShouldSetAmountAndCurrency() {
        Money money = new Money(10, "USD");   // arrange

        // a TestNG Way
        assertEquals(money.getAmount(), 10);  // act and assert
        assertEquals(money.getCurrency(), "USD");

        // a JUnit Way
        AssertJUnit.assertEquals("USD", money.getCurrency());
        AssertJUnit.assertEquals(10, money.getAmount());
    }
}
{%endcodeblock%}

---

## Parameter Tests
Now the first important trick. For many cases, one set of input/output are not sufficient when writing UT. 
{%codeblock lang:java NotSoGoodTest.java %}

public void Test{

    TestObject to1 = new TestObject();
    InputParam input1 = new InputParam();
    OutputParam output1 = new Output();

    assertEquals(to1.calc(input1), output1);

    TestObject to2 = new TestObject();
    InputParam input2 = new InputParam();
    OutputParam output2 = new Output();

    assertEquals(to2.calc(input2), output2);

}

{%endcodeblock %}
If we don't want to write code like this, then *Parameter Test* is our friend. Use a [DataProvider](http://testng.org/javadoc/org/testng/annotations/DataProvider.html) annotation to mark a method as **supplying data** for a test method. It return a 2D array, contains a list of Object[], each Object will be passed as input to the test function.

{% codeblock lang:java MoneyTest.java %}
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;
import static org.testng.Assert.assertEquals;

@Test
public class MoneyTest {

    // Parameter tests
    @DataProvider
     private static final Object[][] getMoney(){
        return new Object[][] {
            {10, "USD"},
            {20, "EUR"},
            {30, "CNY"}
        };
    }

    @Test(dataProvider = "getMoney")
    public void constructorShouldSetAmountAndCurrency(
            int amount, String currency) {
        Money money = new Money(amount, currency);
        assertEquals(money.getAmount(), amount);
        assertEquals(money.getCurrency(), currency);
    }
}
{% endcodeblock %}

See? A DataProvider notation help you define a list of objects. When linked with the dataprovider function, it will automatically pass the object to the test function as parameters. Previous code generate 3 tests.

There are some advance usage of Parameter Tests:

DataProvider could live in another Class so they could be reused (But nee modification to test function, use `@Test(dataProvider = "getMoney", dataProviderClass = DataProviders.class)` annotation).

DataProvider could do lazy initialization, so when you are going to generate many test cases (say 10000+), they don't need to be initialized before the test, incase any cases fail in the middle and it waste a lot of resources. (need further implement an iterator based on DataProvider class)

{% codeblock dataProviderIterator.java %}
@DataProvider(name="colors") 
public Iterator<Object[]> getColors(){
  Set<Object[]> result=new HashSet<Object[]>();
  result.add(new Object[]{"black"});
  result.add(new Object[]{"silver"});
  result.add(new Object[]{"gray"});
  return result.iterator();
}
{% endcodeblock %}

---

## Testing Exceptions
Here is a simple case about how to write UT to test the exception. Some developer might actually include 4 stuffs in a single test (just to improve coverage): 

* start the SUT
* pass an invalid value
* catch the exceptions
* handle it 

However, TestNG provide an much cleaner way to test it. `expectedExceptions` notation are to help.

(Why bother to handle it? Because if we don't handle the exception, then this test will fail...) 

{% codeblock %}
public class MoneyIAETest {
    private final static int VALID_AMOUNT = 5;
    private final static String VALID_CURRENCY = "USD";

    @DataProvider
    private static final Object[][] getInvalidAmount(){
        return new Integer[][] { {-12387}, {-5}, {-1} };
    }

    @Test(dataProvider = "getInvalidAmount",
        expectedExceptions = IllegalArgumentException.class)
    public void shouldThrowIAEForInvalidAmount(int invalidAmount) {
        Money money = new Money(invalidAmount, VALID_CURRENCY);
    }

    @DataProvider
    private static final Object[][] getInvalidCurrency(){
        return new String[][] { {null}, {""} };
    }

    @Test(dataProvider = "getInvalidCurrency",
        expectedExceptions = IllegalArgumentException.class)
    public void shouldThrowIAEForInvalidCurrency(String invalidCurrency) {
        Money money = new Money(VALID_AMOUNT, invalidCurrency);
} }

{% endcodeblock %}

---
### Testing Concurrent Codes

Testing concurrency, to some extent is nightmare. If not correctly implemented, the quality of test might become a problem of a UT. Let's introduce two more attributes which is helpful to test concurrent codes.

* threadPoolSize, which sets the number of threads that are to execute a test method 
* invocationCount, which sets the total number of test method executions.

There is no need for us to implement threads ourselves.

{% codeblock %}

public class SystemIdGenerator implements IdGenerator {
    private static Long nextId = System.currentTimeMillis();

    // is it thread safe?
    public Long nextId() {
        return nextId++;
    }
}
{% endcodeblock %}

{% codeblock lang:java SystemIdGeneratorTest.java %}
public class SystemIdGeneratorTest {
    private IdGenerator idGen = new SystemIdGenerator();

    @Test
    public void idsShouldBeUnique() {
        Long idA = idGen.nextId();
        Long idB = idGen.nextId();
        assertNotEquals(idA, idB, "idA " + idA + " idB " + idB);
    }

    @Test
    public class JVMUniqueIdGeneratorParallelTest {
        private IdGenerator idGen = new SystemIdGenerator();
        private Set<Long> ids = new HashSet<Long>(10000);

        @Test(threadPoolSize = 997, invocationCount = 10000)
        public void idsShouldBeUnique() {
            assertTrue(ids.add(idGen.nextId()));
        }
    }
}
{% endcodeblock %}

I get 307 fail cases in my laptop for the above code. It failed because the unary operator is not atomic and there might be two thread reading the same value of id.

In addition to what we have done, you can also use the timeOut and invocationTimeOut attributes of @Test annotation. Their role is to break the test execution and fail the test if it takes too long (e.g. if your code has caused a deadlock or entered some infinite loop). 

### Collection Testing

* Unitils
* Hamcrest
* FEST Fluent Assertion

#### Unitils
Unitils could help customize the equal definition

{%codeblock lang:java SetEqualityTest.java %}
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import org.unitils.reflectionassert.ReflectionComparatorMode;

import java.util.LinkedHashSet;
import java.util.Set;

import static org.unitils.reflectionassert.ReflectionAssert.assertReflectionEquals;

public class SetEqualityTest {
    // same setA and setB created as in the previous TestNG example

    Set<Integer> setA;
    Set<Integer> setB;

    @BeforeMethod
    public void setUp() {
        setA = new LinkedHashSet<Integer>();
        setB = new LinkedHashSet<Integer>();

        setA.add(1);
        setA.add(2);

        setB.add(2);
        setB.add(1);
    }

    @Test
    public void twoSetsAreEqualsIfTheyHaveSameContentAndSameOrder() {
        // assertReflectionEquals(setA, setB);

        // This will failed with following error
        // --- Found following differences ---
        /*
        [0]: expected: 1, actual: 2
        [1]: expected: 2, actual: 1

        --- Difference detail tree ---
         expected: [1, 2]
           actual: [2, 1]
        */
    }

    @Test
    public void twoSetsAreEqualsIfTheyHaveSameContentAndAnyOrder() {
        assertReflectionEquals(setA, setB,
                ReflectionComparatorMode.LENIENT_ORDER);
    }
}
{% endcodeblock %}

#### Fest Fluent Assertion
FEST Fluent Assertions, which is a part of FEST library, offers many assertions which can simplify collections testing. It also provides a fluent interface, which allows for the chaining together of assertions.

{% codeblock lang:java FestCollectionTest.java  %}
import org.fest.assertions.api.MapAssert;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.LinkedHashSet;
import java.util.Set;

import static org.fest.assertions.api.Assertions.assertThat;
import static org.fest.assertions.api.Assertions.entry;

/**
 * Created by wenzhong on 5/9/15.
 */

public class FestCollectionTest {
    // same setA and setB created as in the previous TestNG example

    Set<Integer> setA;
    Set<Integer> setB;

    @BeforeMethod
    public void setUp() {
        setA = new LinkedHashSet<Integer>();
        setB = new LinkedHashSet<Integer>();

        setA.add(1);
        setA.add(2);

        setB.add(2);
        setB.add(1);
    }

    @Test
    public void collectionsUtilityMethods() {
        assertThat(setA)
            .isNotEmpty()
            .hasSize(2)
            .doesNotHaveDuplicates();

        assertThat(setA).containsOnly(1, 2);
    }

    @Test
    public void mapUtilityMethods() {
        HashMap<String, Integer> map = new LinkedHashMap<String, Integer>();
        map.put("a", 2);
        map.put("b", 3);

        assertThat(map)
            .isNotNull()
            .isNotEmpty()
            .contains(entry("a", 2), entry("b", 3))
            .doesNotContainKey("c");
    }
}
{% endcodeblock %}

FEST provides a fluent interface, which allows for the chaining together of assertions. In this case verification of emptiness, size and the absence of duplicates all gets fitted into a single line of code, but the code is still readable.

---

## Dependency between tests

Now think about a test which want to make sure that user account management is correct. It have 2 functions: adding and deleting user accounts. In traditional UT scope, test are independent so the test code might looks like `testAddition` and `testDeletion`. Note that the addition actually run for 2 times, which actually break the "DRY" rules.

However, testNG use another attribute in @Test notation to solve this problem. This time, an explicit dependency is established between the tests. 

{% codeblock lang:java TestWithDependencyTest.java %}
import org.testng.annotations.Test;

/**
 * Created by wenzhong on 5/9/15.
 */
public class TestWithDependencyTest {

    @Test
    public void testAddition() {
        // adds user X to the system
        // verifies it exists, by issuing SQL query against the database
    }


    @Test
    public void testDeletion() {
        // adds user Y to the system
        // verify that it exists (so later we know that it was actually removed)
        // removes user Y
        // makes sure it does not exist, by issuing SQL query against the database
    }

    @Test(dependsOnMethods = "testAddition")
    public void testDeletion2() {
        // removes user X
        // makes sure it does not exist, by issuing SQL query against the database
    }
}
{% endcodeblock %}

However, it's not always a good thing to have dependency. Think of a test suite with dozens of test, adding a new test will be a nightmare because you might need to find a correct spot to place it into the dependency tree. An alternative is try to initialize the state before each test. And use the dependency in integration test and E2E test.

### Private Method Testing
Should we test private method? Of course. But from the best practice we should test it via public method test. However, when facing legacy code, we might face a dilemma: we need test to make sure it work as expect, but writing test need refactor on code, without test we don't know whether our refactor is correct.

So when facing legacy code, we compromise on private method testing.

#### Built-in support in Java
Let's examine some native support from Java.

{% codeblock %}

public class ClassWithPrivateMethod {
    private boolean privateMethod(Long param) {
        return true;
    }
}

import org.testng.annotations.Test;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

import static org.testng.Assert.*;

public class ClassWithPrivateMethodTest {
    @Test
    public void testingPrivateMethodWithReflection()
            throws NoSuchMethodException, InvocationTargetException,
                   IllegalAccessException {

        // Note: this is an ugly implementation

        ClassWithPrivateMethod sut = new ClassWithPrivateMethod();

        Class[] parameterTypes = new Class[1];
        parameterTypes[0] = java.lang.Long.class;

        Method m = sut.getClass()
            .getDeclaredMethod("privateMethod", parameterTypes);

        // make it accesible outside of class
        m.setAccessible(true);

        Object[] parameters = new Object[1];
        parameters[0] = 5569L;

        // actually invoke
        Boolean result = (Boolean) m.invoke(sut, parameters);
        assertTrue(result);
    }
}

{% endcodeblock %}

#### Using PowerMock

[PowerMock](https://code.google.com/p/powermock/wiki/MockPrivate) give us a cleaner approach to work on testing private methods. The [WhiteBox](http://powermock.googlecode.com/svn/docs/powermock-1.3.7/apidocs/org/powermock/reflect/Whitebox.html) class provide various utilities for accessing internals of a class. Basically it's a simplified reflection utility intended for tests.
{% codeblock %}

import org.powermock.reflect.Whitebox;

public class ClassWithPrivateMethodTest {

    @Test
    public void testingPrivateMethodWithReflectionByPowerMock()
            throws Exception, IllegalAccessException {
        ClassWithPrivateMethod sut = new ClassWithPrivateMethod();
        
        Boolean result = Whitebox
            .invokeMethod(sut, "privateMethod", 302483L);
        assertTrue(result);
    }

}
{% endcodeblock %}

### Testing non dependency injection code
Think about the following production code (which is less test-able), how can we mock the `MyCollaborator` class?

{% codeblock %}
public class MySut {
    public void myMethod() {
        MyCollaborator collaborator = new MyCollaborator();
        // some behaviour worth testing here which uses collaborator
    }
}
{% endcodeblock %}

PS. the correct way with dependency injection should be
{% codeblock %}
public class MySut {
    public void myMethod(MyCollaborator collaborator) {
        // some behaviour worth testing here which uses collaborator
    }
}

{% endcodeblock %}

#### PowerMock to Rescue

{% codeblock %}

import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.testng.IObjectFactory;
import org.testng.annotations.ObjectFactory;
import org.testng.annotations.Test;

import static org.powermock.api.mockito.PowerMockito.mock;

@PrepareForTest(NotDOCInjectedSUT.class)
public class NonInjectedDOCTest{
    @ObjectFactory
    public IObjectFactory getObjectFactory() {
        return new org.powermock.modules.testng.PowerMockObjectFactory();
    }

    @Test
    public void testMyMethod() throws Exception {
        NotDOCInjectedSUT sut = new NotDOCInjectedSUT();
        MyCollaborator collaborator = mock(MyCollaborator.class);

        // the whenNew function applies to
        // normal test using Mockito's syntax
        // e.g. Mockito.when(collaborator.someMethod()).thenReturn(...)
        PowerMockito.whenNew(MyCollaborator.class)
                .withNoArguments().thenReturn(collaborator);
    }
}

{% endcodeblock %}

Here some new attribute and method are introduced:
* the `@PrepareForTest` annotation informs PowerMock that the `NotDOCInjectedSUT` class will create a new instance of some other class. In general, this is how PowerMock learns, about which classes it should perform some bytecode manipulation.
* In order to use PowerMock with TestNG, we need to make PowerMock responsible for the creation of all of the test instances. So we use the `@ObjectFactory` notation to assign PowerMockObjectFactory as the Factory class to generate test instances.
* The test double is created as usual - with the static mock() method of Mockito.
* `whenNew()` is the place magic happens: whenever a new object of the MyCollaborator class gets created, our test double object (collaborator) will be used instead. Two of PowerMock’s methods - whenNew() and withNoArguments() - are used to control the execution of a no-arguments constructor of the MyCollaborator class.
* Note that `static` methods could also be mocked as the `new` operator.

#### ArgumentCaptor
But if the `collaborator` class have some input parameters for its constructor?

Here we have a not quite good class -- `PIM` (Personal Information Manager?), and related class `Calendar` and `Meeting`.

{% codeblock %}
public interface Calendar {
    public void addEvent(Event event);
}

public class Meeting implements Event {
    private final Date startDate;
    private final Date endDate;
    public Meeting(Date startDate, Date endDate) {
        this.startDate = new Date(startDate.getTime());
        this.endDate = new Date(endDate.getTime());
    }
    public Date getStartDate() {
        return startDate;
    }
    public Date getEndDate() {
        return endDate;
    }
}

public class PIM {
    private final static int MILLIS_IN_MINUTE = 60 * 1000;
    private Calendar calendar;
    public PIM(Calendar calendar) {
        this.calendar = calendar;
    }
    public void addMeeting(Date startDate, int durationInMinutes) {
        Date endDate = new Date(startDate.getTime() + MILLIS_IN_MINUTE * durationInMinutes);

        Meeting meeting = new Meeting(startDate, endDate);
        calendar.addEvent(meeting);
    }
}

{% endcodeblock %}

As we could see that the meeting inside the `addMeeting` method is hard to mock. However, Mockito provide a `ArgumentCaptor` function, which we could use to get information from the type `Meeting`.

{% codeblock %}
import org.mockito.ArgumentCaptor;
import org.testng.annotations.Test;

import java.util.Date;

import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.verify;
import static org.testng.Assert.*;

/**
 * Created by wenzhong on 5/10/15.
 */

public class PIMTest {
    private static final int ONE_HOUR = 60;
    private static final Date START_DATE = new Date();
    private static final int MILLIS_IN_MINUTE = 1000 * 60;
    private static final Date END_DATE
            = new Date(START_DATE.getTime() + ONE_HOUR * MILLIS_IN_MINUTE);

    @Test
    public void shouldAddNewEventToCalendar() {
        Calendar calendar = mock(Calendar.class);
        PIM pim = new PIM(calendar);

        // An object of the ArgumentCaptor class is created, 
        // which will gather information on arguments of the type Meeting.
        ArgumentCaptor<Meeting> argument
                = ArgumentCaptor.forClass(Meeting.class);

        pim.addMeeting(START_DATE, ONE_HOUR);

        // The addEvent() method’s having been called is verified,
        // and Mockito is instructed to capture arguments of this method call.
        verify(calendar).addEvent(argument.capture());

        Meeting meeting = argument.getValue();
        assertEquals(meeting.getStartDate(), START_DATE);
        assertEquals(meeting.getEndDate(), END_DATE);
    }
}

In the above code, the actual argument to the `addEvent()` method is extracted from the `ArgumentCaptor` object.

{% endcodeblock %}

---
# Writing Testable Code

Code that wasn’t designed to be testable is not testable.

### Rely on Dependency Injection
So every dependency could be mocked.

### Never hide a TUF within TUC
*TUF*, or *Test UnFriendly Feature*, include following examples:

* Database access
* FileSystem access
* Network access
* Affect of side effect access
* lengthy / inscrutable computations
* static variable usage 

*TUC*, or *Test Unfriendly Constructor*, include following examples:
* Final methods
* Final classes
* Static methods
* Private methods
* Static InitializationExpressions
* Constructors
* ObjectInitialization Blocks
* New-Expressions

### Handling existing issues
1. subclass and override

{% codeblock %}
public class EventResponder
{
    void showNotification(String notificationMessage) {
        JOptionPane.showMessageDialog(null, notificationMessage);
    }

    public void respond() {
        ...
        showNotification(message);
    }
}
{% endcodeblock %}

{% codeblock testRespond %}
public class TestingEventResponder extends EventResponder
{
    void showNotification(String notificationMessage) {
    }
}
{% endcodeblock %}

Let's think about when we could not use this "subclass and override" way to test?
* Final method (we could not override)
* Final class (so we could not inherit)
* Static method (we could not override)
* Private method (we could not override)
* Constructor (they have to run and we have no way to override it)

Strictly speaking, final classes and final methods are only a problem in testing when they hide a TUF, but it's nice to have a carefully considered reason for using them rather than just using them by default.

# References & Further Reading
* http://martinfowler.com/bliki/FluentInterface.html
* [Testable Java](http://www.objectmentor.com/resources/articles/TestableJava.pdf) by Michael Feathers
* [Next Generation Java Testing: TestNG and Advanced Concepts](http://testng.org/doc/book.html) By Cédric Beust, Hani Suleiman
* [Working Effectively With Legacy Code](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052) by Michael Feathers
* [Refactoring: Improving the Design of Existing Code](http://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672/) by Martin Fowler, Kent Beck, John Brant, William Opdyke, Don Roberts
* [Growing Object-Oriented Software, Guided by Tests](http://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627/) by Steve Freeman , Nat Pryce 
* [Java Concurrency in Practice](http://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601/)
* [Clean Code: A Handbook of Agile Software Craftsmanship](http://www.amazon.com/gp/product/0132350882martin2008) By Robert C. Martin
