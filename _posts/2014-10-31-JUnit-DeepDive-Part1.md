---
layout: post
category : pages
tags : [Java, JUnit, Testing]
disqusid : junit_deepdivepart1
---

Most of the Java developers know JUnit for many years and probably use it on a daily basis. But like all things
JUnit evolves and so should we. Especially if your test code lacks in view of readability, understandability and clean code
you should have a closer look at some of the newer features of JUnit.
The upcoming blog series starting with this article covering the JUnit basics
will take a close look on the internals, concepts and best practices of JUnit nowadays.<br/>

JUnit (current version 4.11) itself is a test framework which provides some annotations, interfaces and utility classes
which can be used to write test classes, lifecycle methods and test methods that will ensure the correctness of your business code.
It is widely known and used and plays a big role when it comes to continuous integration and agile projects.<br/>

<h2>Test classes</h2>

A test class is just a plain java class and nothing more. In view of JUnit it is managed through a <i>JUnitRunner</i> which normally provides
a lifecycle.
Like many framework nowadays JUnit follows the CoC (Convention over Configuration)
which basically means that you only have to declare things that are not part of the convention. That way your code keeps small and
therefore is more readable and understandable. Considering JUnit test classes it means that you can provide your own <i>JUnitRunner</i> but JUnit already
provides a default <i>JUnitRunner</i> (internally mapped to <i>BlockJUnit4ClassRunner</i>). This default <i>JUnitRunner</i> inherits from <i>
                                                ParentRunner</i> which provides you
                                               a lifecycle which is shown in the following image and will be explained further in this article.
<img src="{{site.baseurl}}/public/images/2014-10-31-JUnitLifecycle_simple.png"/>
But keep in mind that this lifecycle represents only the default lifecycle. If you develop your own <i>JUnitRunner</i> or using other JUnit-Rules it can be something really different.
But those topics i will cover in another blog post.<br/>

<h2>Lifecycle methods</h2>
A lifecycle method here is a method which is marked with one of the following annotations and can be used to configure your tests.
<table border="1">
    <tr>
        <th>Annotation</th>
        <th>Explanation</th>
    </tr>
    <tr>
        <td>@BeforeClass</td>
        <td>Static methods (can occur multiple times) to initialize things on the test class level. The order is dependent on the
        underlying JVM</td>
    </tr>
    <tr>
        <td>@AfterClass</td>
        <td>Static methods (can occur multiple times) to clean up things on the test class level. The order is dependent on the
                                                                                                         underlying JVM</td>
    </tr>
     <tr>
            <td>@Before</td>
            <td>Non Static methods (can occur multiple times) to initialize things on the test method level. The order is dependent on the
            underlying JVM</td>
        </tr>
        <tr>
            <td>@After</td>
            <td>Non Static methods (can occur multiple times) to clean up things on the test method level. The order is dependent on the
            underlying JVM</td>
        </tr>
</table>
As already mentioned above this is just the half truth because since JUnit 4.7 the concept of rules are integrated.<br/>

One important point: As you can see in the lifecycle diagram above both (the constructor and <i>@Before</i> annotated methods) are executed before each
test method and therefore seem to have the same semantic (at least
from the test-method point of view). But there are some important differences to keep in mind:
<ul>
    <li>It breaks the symmetry between <i>@Before</i> and <i>@After</i></li>
    <li>If the constructor throws an exception <i>@After</i> is not
        executed
    </li>
    <li>JUnit can be used with several other frameworks (e.g.
        Mockito) which rely on the lifecycle at least partially. Therefor using the
        constructor for initializing can lead for example to <i>NullPointerExceptions</i>.
    </li>
</ul>
Looking at that it is reasonable to say prefer <i>@Before</i> to the constructor.<br/>

One historical note: The usage of annotations is available since JUnit 4.0. Before that you had to follow a naming convention (e.g. test methods
has to start with "test"). <br/>

<h2>Test methods</h2>
A test method for JUnit is a non static, public method which is marked with a <i>@Test</i> annotation. Such a method should contain so
called assertions and/or assumptions (yes - there are use cases with none of them). The difference between them is that a cause of an assertion leads
to a fail of the test method because an AssertionError is thrown. Indeed an assumption also throws an exception (AssumptionViolatedException) but this one
marks the test just as ignored. So an assumption has more the semantic of a useless test which could be the case if you rely on another system
which is not running or cannot be started.<br/>
The following table shows the basic assertions which are part of the <i>Assert</i> class.
<table border="1">
    <tr>
        <th>Assert</th>
        <th>Explanation</th>
    </tr>
    <tr>
        <td>assertTrue</td>
        <td>Checks whether a given condition is true. Fails if condition
            is false. Example: assertTrue("User not signed up",user.isSignUp())
        </td>
    </tr>
    <tr>
        <td>assertFalse</td>
        <td>Opposite of assertTrue</td>
    </tr>
    <tr>
        <td>assertNull</td>
        <td>Checks whether a given object is null. Fails if object is
            not null. Example: assertNull(calculator)
        </td>
    </tr>
    <tr>
        <td>assertNotNull</td>
        <td>Opposite of assertNull</td>
    </tr>
    <tr>
        <td>assertSame</td>
        <td>Checks whether two objects are reference the same object.
            Fails if not. Example: assertSame("Hallo Welt", "Hallo " + "Welt")
        </td>
    </tr>
    <tr>
        <td>assertNotSame</td>
        <td>Opposite of assertSame. Example: assertNotSame("Hallo Welt",
            new String("Hallo Welt"))
        </td>
    </tr>
    <tr>
        <td>assertEquals</td>
        <td>Checks whether two objects are equal. Example:
            assertEquals("Hallo Welt", new String("Hallo Welt"))
        </td>
    </tr>
    <tr>
        <td>assertNotEquals</td>
        <td>Opposite of assertEquals</td>
    </tr>
    <tr>
        <td>assertArrayEquals</td>
        <td>Checks whether two arrays are identical in length and
            elements. Example: assertArrayEquals(new int[]{1,2}, new int[]{1,2})
        </td>
    </tr>
    <tr>
        <td>assertThat</td>
        <td>Makes use of a matcher which describes itself to give
            feedback if it fails. Example: assertThat(new String("Hallo Welt"),
            equalTo("Hallo Welt"))
        </td>
    </tr>
    <tr>
        <td>fail</td>
        <td>Throws an <i>AssertionError</i> so that the test will fail
        </td>
    </tr>
</table>
Also note that every assert method is overloaded with an additional errorMessage
String in front of the parameter list. This is considered best practice because it makes the assertion or assumption more
readable and significant if something goes wrong (FYI: There is also an explicit
                                                                       PMD-Rule <i>JUnitAssertionsShouldIncludeMessage</i> for that).<br/>

<table border="1">
    <tr>
        <th>Assume</th>
        <th>Explanation</th>
    </tr>
    <tr>
        <td>assumeTrue</td>
        <td>Checks whether a given condition is true. Skips test and marks it as ignored if condition
            is false. Example: assumeTrue("User not signed up",user.isSignUp())
        </td>
    </tr>
    <tr>
        <td>assumeFalse</td>
        <td>Opposite of assumeTrue</td>
    </tr>
    <tr>
        <td>assumeNotNull</td>
        <td>Checks whether one or many given objects are not null. Skips test and marks it as ignored otherwise. Example: assertNull(calculator)
        </td>
    </tr>
    <tr>
        <td>assumeNoException</td>
        <td>Checks whether a given <i>Throwable</i> is not thrown. Otherwise test is skipped and marked as ignored.</td>
    </tr>
    <tr>
        <td>assumeThat</td>
        <td>Makes use of a matcher to check if a condition is true. If not test is skipped and marked as ignored.
        </td>
    </tr>
</table>

At this point i left out the following points because they would blow up this article. So stay tuned and read more
in one of the following topics:
<ul>
<li><i>assertThat</i> with the big topic <i>Matcher</i></li>
</ul>

<h2>Example</h2>
So far i had only covered theory. Now i am giving a simple example which uses annotations.
The calculator here is specialized for integers but could be easily extended for other numeric types. Our functional methods
would be add, subtract, multiply and divide.

	{% highlight java linenos%}
public class Calculator {

	public int add(int a, int b) {
		return a + b;
	}

	public int subtract(int a, int b) {
		return a - b;
	}

	public int multiply(int a, int b) {
		return a * b;
	}

	public int divide(int a, int b) {
		return a / b;
	}
}{% endhighlight%}
So far nothing special so let us have a look at the test. I created a test method for each public method of the
calculator and used the
<i>@Before</i> annotated method to initialize the calculator. That way each time before the test method is executed the calculator is newly initialized. The reason for
that is quite simple - i want to eliminate possible side effects in the future (which could occur if i would use <i>@BeforeClass</i>)
and do not want to violate the DRY (Don't Repeat Yourself) principle (which would occur if i would initialize the calculator in every test
method).
<br/>
	{% highlight java linenos%}
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.fail;

import org.junit.Before;
import org.junit.Test;

public class CalculatorSimpleTest {

	Calculator calculator;

	@Before
	public void setUp() throws Exception {
		calculator = new Calculator();
	}

	@Test
	public void testAdd() {
		String errorMessage = "Adding failed";
		int expected = 3;
		int add = calculator.add(1, 2);
		assertEquals(errorMessage, expected, add);

		// Bad assertTrue(3==calculator.add(1, 2));
		// Bad assertSame(3, calculator.add(1, 2));
	}

	@Test
	public void testSubtract() {
		String errorMessage = "Subtracting failed";
		int expected = -1;
		int add = calculator.subtract(1, 2);
		assertEquals(errorMessage, expected, add);
	}

	@Test
	public void testMultiply() {
		String errorMessage = "Multiplying failed";
		int expected = 2;
		int add = calculator.multiply(1, 2);
		assertEquals(errorMessage, expected, add);
	}

	@Test
	public void testDivide() {
		String errorMessage = "Dividing failed";
		int expected = 0;
		int add = calculator.divide(1, 2);
		assertEquals(errorMessage, expected, add);
	}
}
{% endhighlight%}
As you can see every test method checks an expected value against an
actual computation result. If there is a mismatch an assertion error is
thrown. <br/>
If something went wrong (here i changed the expected result of the multiply method) you can see corresponding error message.
<br/>
<img src="{{site.baseurl}}/public/images/2014-10-31-message.png"/>

<h2>ExceptionHandling</h2>
Now you are able to develop standard tests and in many cases this
should be sufficient (for the moment :)). But in view of testing
exceptions your test classes can suffer really fast. Considering the
calculator you can see easily that at least the divide method needs one
more test.
	{% highlight java linenos%}
	@Test
	public void testDivide_WithZero_WillFail() {
		assertEquals(0, calculator.divide(1, 0));
	}
	{% endhighlight%}
This test will fail because an
<i>ArithmeticException</i> is thrown. But lets assume that this
behavior (throwing the exception) is correct because i want the
client handle the exception. So with a naive small refactoring i can get
this.

	{% highlight java linenos%}
	@Test
	public void testDivide_WithZero_WillFail() {
		try {
			assertEquals(0, calculator.divide(1, 0));
			fail();
		} catch (ArithmeticException e) {
			// Bad
		}
	}
	{% endhighlight%}
Ok, now i have tested the correct behavior but the resulting test code
does not look good. I have boiler-plate code and doubled the size of my previous test code therefore it is less
understandable. A better way to do that is using the
<i>@Test</i> annotation the following way.
	{% highlight java linenos%}
	@Test(expected = ArithmeticException.class)
	public void testDivide_WithZero_WillFail() {
		assertEquals(0, calculator.divide(1, 0));
	}
	{% endhighlight%}
This test method only fails if the
<i>ArithmeticException</i> is not thrown.<br/>

So this is what i consider as the JUnit basics but there will be more in the upcoming articles.<br/>
Have fun coding.