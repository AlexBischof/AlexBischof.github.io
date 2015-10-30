---
layout: post
category : pages
tags : [Java, JUnit, Testing]
---
It's been a while since the last DeepDive article but i was quite busy during the year and it looks like 
[junit-lambda](http://junit.org/junit-lambda.html) project is picking up pace. So it is really time to catch up and have some deep dive...

This time it is all about *JUnit Rules* which are basically just another way to interfere with the JUnit lifecycle. In detail you can look at
 it as an extension of the *@BeforeClass/@AfterClass/@Before/@After* mechanism which had led to some problems or better *codesmells*. 
 Within the lifecycle *Rules* are always executed before/after the *old* lifecycle hooks as you can see in the diagram below. 
 
![Lifecycle](/public/images/2015-10-30-junitlifecycle.jpg "JUnit Lifecycle with Rules")

Considering the lifecycle you might have noticed that there are two types of rules: **ClassRules** and **Rules** where the former one is 
for static context (e.g. @BeforeClass/@AfterClass) and the latter one for the testmethod context (e.g. @Before/@After). The good news is
that the distinction is not part of the implementation (or more precise doesn't have to). But let us have a look at an example:

Assume a hypothetical service which checks if a file exists and you want to have a test for that. With JUnit *Rules* it would look 
like that.

{% highlight bash linenos%}
public class MyServiceTest {
 
     @Rule
     public TemporaryFolder temporaryFolder = new org.junit.rules.TemporaryFolder();
 
     @Test
     public void testFileExists() throws IOException {
         assertTrue(new MyService().checkFileExists(temporaryFolder.newFile()));
     }
 }
 
 class MyService {
     public boolean checkFileExists(File file) {
         return file.exists();
     }
 }
{% endhighlight %}

Basically you have to take care of three things:

 * Add **@Rule** to a rule member
 * that member has to be **public**
 * the rule has to be initialized
 
The example above makes use of a standard rule. To change that test to a classrule you have to change @Rule to **@ClassRule** and
add the **static** modifier to the member.

{% highlight bash linenos%}
public class MyServiceTest {
 
     @ClassRule
     public static TemporaryFolder temporaryFolder = new org.junit.rules.TemporaryFolder();
 
     @Test
     public void testFileExists() throws IOException {
         assertTrue(new MyService().checkFileExists(temporaryFolder.newFile()));
     }
 }
{% endhighlight %}

#### Provided/Custom Rules

For most of the standard problems JUnit already provides rules 

 * DisableOnDebug
 * ErrorCollector
 * ExpectedException
 * ExternalResource - base class for rules with a setup and a teardown
 * RuleChain
 * Stopwatch
 * TemporaryFolder
 * TestName - makes the testmethod name available
 * TestRule
 * TestWatcher - base class for rules which makes use of the test outcome without modifying it
 * Timeout
 * Verifier - base class for rules which want to change the outcome of tests (like ErrorCollector)

But this is sometimes not enough.
To implement a custom *Rule* you have to create a class which **implements TestRule** and overwrite the **apply** method. 
Within that method create a new statement and call **base.evaluate** and that is all. 

{% highlight bash linenos%}
public class MyRule implements TestRule {
    @Override public Statement apply(Statement base, Description description) {
        return new Statement() {
            @Override public void evaluate() throws Throwable {
                //before
                base.evaluate();
                //after
            }
        };
    }
}
{% endhighlight %}

Note: A custom rule also can be created with implementing *MethodRule* which is kind of replaced with *TestRule* in JUnit 4.9.
The signature of the *apply*-method is different especially the *FrameworkMethod* parameter which can be useful if you want
to get the test method name. But it also has the drawback that you cannot use that rule as an classrule.

### RuleChains

Like lifecycle hooks (e.g. @Before) it is also possible to have multiple rules within one test class. 
But what if you need to executed them in a specific order?

The solution is a *RuleChain* which itself is also a rule 
{% highlight bash linenos%}
@Rule
public RuleChain chain= RuleChain
  	                       .outerRule(new LoggingRule("outer rule")
  	                       .around(new LoggingRule("middle rule")
  	                       .around(new LoggingRule("inner rule");
{% endhighlight %}
which leads to the following output:
{% highlight bash linenos%}
starting outer rule
starting middle rule
starting inner rule
finished inner rule
finished middle rule
finished outer rule
{% endhighlight %}

### Errorhandling

As we are in a testing context where also exceptions can be thrown it is always interesting what happens in the specific lifecycle
phases when an exception is thrown. The diagram below gives an overwiew over that topic. The only surprise for me was that the after
part of a class rule is not executed if an exception is thrown in the @BeforeClass method.

![Lifecycle](/public/images/2015-10-30-junitlifecycleerrors.jpg "JUnit Lifecycle with Rules")

### Conclusion

From my point of view JUnit *Rules* are a great way to make your test code cleaner. In particular it is very useful
 for cross cutting concerns (e.g. external resources, logging) and enhances the *Inversion of Control* design. The already
 provided rules should cover most of the use cases and even if not writing a custom rule is no rocket science.
 On the other hand it enlarges the *JUnit* universe (e.g. runners, lifecycle hooks, theories) and you need to know more
 of there interaction between each other.
 
 
 
 