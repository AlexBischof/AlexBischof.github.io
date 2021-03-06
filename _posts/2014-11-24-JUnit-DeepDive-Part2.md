---
layout: post
category : pages
tags : [Java, JUnit, Testing]
---

This time i want to cover the topic of aggregating JUnit tests
 which is also part of my blog series <i>Is your JUnit knowhow up to date?</i> The last article can be found
 <a href="http://www.rapidpm.org/2014/10/31/is-your-junit-knowhow-up-to-date-part-.html">here</a>. So let us get started.

<h3>Suites</h3>

A JUnit <i>Suite</i> is the simplest and oldest way to aggregate JUnit tests. You can use it by annotating your test suite
with <i>@Suite</i> as you can see in the following code snippet:<br/>

{% highlight java linenos%}
@RunWith(Suite.class)
@Suite.SuiteClasses({CalculatorSimpleTest.class, CalculatorPerformanceTest.class})
public class CalculatorSuite {
}
{% endhighlight %}
Basically you tell JUnit to look out for the class array of the <i>@Suite.SuiteClasses</i> which contains all the classes of
 the suite. But there is a little bit more. <i>Suite</i> itself inherits from <i>ParentRunner</i> which i covered in the last
 <a href="http://www.rapidpm.org/2014/10/31/is-your-junit-knowhow-up-to-date-part-.html">article</a>.
Therefore <i>Suites</i> have a lifecycle but in comparison to the default runner only include the <i>Class Ready</i> lifecycle methods
which are:<br/>
<ul>
<li><i>@BeforeClass</i></li>
<li><i>@AfterClass</i></li>
<li><i>Class Rules (i have not covered it yet)</i></li>
</ul>
But although it is technically possible to used them it does not mean that it is good. In general a suite class should only act as an
aggregation container. One reason for that is that suite classes can be listed in suites and categories itself so that you can build up
hierarchical test structures. If you now use lifecycle methods in your suites you probably decrease understandability and your isolation
between your suites.<br/><br/>

I also took a deep look at the <i>Suite.SuiteClasses</i> annotation which i considered at first as a bad solution to the problem of aggregating test classes
and suites. The reason for that was the declarative nature of it which means that you have to declare every single class or suite. Even with IDE support nowadays this
could lead to some work if you have many classes. Considering bug fixes or enhancements of a system there is also the problem that those new tests maybe do not make
it in the test suites and therefore could lead to bad reports about the health or quality of the aggregation part.<br/>
On the second look i changed my mind. The first reason for that was that there are already some libraries with whom you can overcome the declarative nature problem. The
two most promising ones should be:<br/>
<ul>
<li><a href="http://johanneslink.net/projects/cpsuite.jsp">ClasspathSuite</a> - As the name suggests this library extends the suite idea for classpaths.</li>
<li><a href="https://code.google.com/p/junit-toolbox/">JUnit Toolbox</a> - A JUnit extension library which provides among other things
a WildcardPatternSuite which extends the suite idea for wildcards.</li>
</ul>
The second reason came after thinking about when aggregate tests anyway which i would do for the following topics:<br/>
<ul>
<li>by domain or package</li>
<li>by test stage (for example component, integration or performance tests)</li>
</ul>
There are probably more topics by which you can aggregate your tests. The point is that in such cases you explicitly want to declare which tests are in that suite and which
are not. <br/>

FYI: In the old days of JUnit 3 <i>suites</i> were recognized by a <i>public static Test suite()</i> method in which you had to add the test classes.
<br/><br/>

<h3>Categories</h3>

A more flexible way to aggregate tests are <i>Categories</i> which were introduced with JUnit 4.8 and strangely enough are still in the experimental package.
The basic concept is similar to suites which means that there is a <i>Categories</i> class which is a JUnit runner (in detail inherits from <i>Suite</i>) and has to be declared with <i>@RunWith(Categories.class)</i>
on top of your aggregation container. You also have to declare your test classes with <i>@Suite.SuiteClasses</i>.<br/> The new part is that you can mark your test classes and/or test methods
with <i>@Category</i> and a marker class which is used as a filter in your test aggregation container. For that there are two more annotations <i>@Categories.IncludeCategory</i>
and <i>@Categories.ExcludeCategory</i> which take a category filter class (the default behavior, without any include or exclude, includes all test methods).
The only limitation i found so far is that you can use only one category marker class on the test aggregation container and that you can not
repeat those annotations.<br/>
An example of an aggregate container would be look like this:<br/>

{% highlight java linenos %}
@RunWith(Categories.class)
@Categories.IncludeCategory(SlowTests.class)
@Suite.SuiteClasses( { CalculatorSimpleTest.class, CalculatorPerformanceTest.class})
public class OnlySlowTestSuite {}

public interface SlowTests{}
{% endhighlight %}

In this example i had declared a category suite which uses the test classes <i>CalculatorSimpleTest</i> and <i>CalculatorPerformanceTest</i> and include from that
classes only the test methods which are annotated with <i>@Category(SlowTests.class)</i>. A <i>@Category</i> annotation can be used on class and/or on method level and
expects an array of classes. In general it is recommended to use only one class because otherwise it could be really difficult
to understand your suite filters considering including and excluding.
<br/>
In the example above i declare the <i>SlowTests</i> category class marker.

{% highlight java linenos %}
public class CalculatorSimpleTest {

    //snip..

    @Category(SlowTests.class)
    @Test
    public void testSubstract() {
        for (int i = 0; i < 10000; i++) {
            try {Thread.sleep(10l);} catch (InterruptedException e) {
                e.printStackTrace();
            }

            String errorMessage = "Substracting failed";
            int expected = 1;
            int add = calculator.substract(i + 1, i);
            assertEquals(errorMessage, expected, add);
        }
    }
}
{% endhighlight %}

If you want to skip all slowtests you would exchange <i>IncludeCategory</i>
with <i>ExcludeCategory</i>.<br/><br/>
At the time of writing this article JUnit 4.12 beta3 is released which already denotes a little change in the lifecycle handling
considering categories. Until now it was possible to use <i>@Category</i> on lifecycle methods like <i>@Before</i> which made
those tests very hard to understand. Therefore JUnit introduced a validation handling which prohibits this usage.<br/><br/>
