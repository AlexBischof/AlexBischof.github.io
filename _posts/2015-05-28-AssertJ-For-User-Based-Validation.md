---
layout: post
category : pages
tags : [Java, AssertJ, Testing]
---

Usually [AssertJ](http://assertj.org) is a pure testing framework and therefore should have *test* scope in our 
*pom.xml* or *build.gradle*. But recently i had facing a problem where my first thought was *AssertJ* and it was not in the test scope ;).
So this article contains a PoC and this were the requirements:

1. The system should compare the values of a pojo (only numbers) against a reference pojo with a configurable strategy. This 
comparison can be an exact one as well as a close to compare with a given (also configurable) offset. 
2. Thereby the implementation should collect all failing values in contrast to only the first one.
3. And the implementation should provide a detailed failure message which means it should contain the actual and the expected
value. With that users can evaluate those messages and propabliy adjust offsets for example.

*As a side note: The code should be implemented as easy and readable as possible. ;)*

### Solution

The crucial part for me here is the [AssertJ Assertions Generator](http://joel-costigliola.github.io/assertj/assertj-assertions-generator.html)
which generates a **PojoAssert** class. This class contains *hasXXX* and *isCloseToXXX* methods which fits the requirements. Additionally i configured the plugin that way
that it also generates the **SoftAssertions** class which collects all failures. So requirement 2 is also met...two to go

*pom.xml*
{% highlight bash linenos%}
<!-- snip -->
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>2.0.1-SNAPSHOT</version>
        </dependency>
<!-- snip -->
        <plugins>
            <plugin>
                <groupId>org.assertj</groupId>
                <artifactId>assertj-assertions-generator-maven-plugin</artifactId>
                <version>1.6.1-SNAPSHOT</version>
                <configuration>
                    <classes>
                        <param>de.bischinger.validation.model.MyPojo</param>
                    </classes>

                    <targetDir>src/main/java</targetDir>
                    <generateAssertions>false</generateAssertions>
                    <generateBddAssertions>false</generateBddAssertions>
                    <generateSoftAssertions>true</generateSoftAssertions>
                    <generateJUnitSoftAssertions>false</generateJUnitSoftAssertions>
                </configuration>
            </plugin>
        </plugins>
{% endhighlight %}

*Note: If you wonder about the SNAPSHOT versions, the isCloseToXXX methods are currently not in version 1.6.0 but will come soon.*

If you take a close look at the generated failure messages of the *AssertJ* assertions you can see that every AssertionError contains the expected as well
as the actual values. If you are not satisfied with that you also have the possibility to override that message to your needs. 
Only one requirement to go...

*Failure message example with two failures*
{% highlight bash linenos%}
The following 2 assertions failed:
1) 
Expecting sum2:
  <2>
to be close to:
  <-5>
by less than <1> but difference was <7>
2) 
Expecting sum4:
  <4>
to be close to:
  <10>
by less than <1> but difference was <6>
{% endhighlight %}

The configurable part can be done with a *Strategy* design pattern which takes the actual and expected pojo as well as indexed based offsets. 
This example provides one strategy for *exact* checks and one strategy for *is close to* checks where as the latter one takes use of the offsets.

{% highlight bash linenos%}
@FunctionalInterface
public interface ValidationStrategy {

    void validate(MyPojo actual, MyPojo expected, long[] offset);

    //Offset ignored here
    ValidationStrategy VALIDATE_ALWAYS_EQUAL = (a, e, o) -> {
        SoftAssertions softAssertions = new SoftAssertions();
        softAssertions.assertThat(a).hasSum1(e.getSum1());
        softAssertions.assertThat(a).hasSum2(e.getSum2());
        softAssertions.assertThat(a).hasSum3(e.getSum3());
        softAssertions.assertThat(a).hasSum4(e.getSum4());

        softAssertions.assertAll();
    };

    ValidationStrategy VALIDATE_BY_OFFSETMAPPING = (a, e, o) -> {
        SoftAssertions softAssertions = new SoftAssertions();
        softAssertions.assertThat(a).hasCloseToSum1(e.getSum1(), o[0]);
        softAssertions.assertThat(a).hasCloseToSum2(e.getSum2(), o[1]);
        softAssertions.assertThat(a).hasCloseToSum3(e.getSum3(), o[2]);
        softAssertions.assertThat(a).hasCloseToSum4(e.getSum4(), o[3]);

        softAssertions.assertAll();
    };
}
{% endhighlight %}

{% highlight bash linenos%}
public class ValidationStrategyTest {

    private static MyPojo actual;

    @BeforeClass
    public static void before() {
        actual = new MyPojo(1, 2, 3, 4);
    }

    @Test
    public void should_pass_EqualsStrategy_when_everything_is_equal() throws Exception {
        ALWAYS_EQUAL.validate(actual, actual, null);
    }

    @Test(expected = SoftAssertionError.class)
    public void should_fail_EqualsStrategy_with_non_equal() throws Exception {
        ALWAYS_EQUAL.validate(actual, new MyPojo(1, 2, 0, 3), null);
    }

    @Test
    public void should_pass_OffsetStrategy_when_everything_is_equal() throws Exception {
        BY_OFFSETMAPPING.validate(actual, actual, 0, 0, 0, 0);
    }

    @Test
    public void should_pass_OffsetStrategy_when_everything_is_closeTo() throws Exception {
        BY_OFFSETMAPPING.validate(actual, new MyPojo(0, 3, 2, 5), 1, 1, 1, 1);
    }

    @Test(expected = SoftAssertionError.class)
    public void should_fail_OffsetStrategy_when_something_is_not_closeTo() throws Exception {
        BY_OFFSETMAPPING.validate(actual, new MyPojo(0, -5, 2, 10), 1, 1, 1, 1);
    }
}
{% endhighlight %}
   
If you are interested in the PoC you can find the sources on my [Github-Repository](https://github.com/AlexBischof/validationstrategy).




