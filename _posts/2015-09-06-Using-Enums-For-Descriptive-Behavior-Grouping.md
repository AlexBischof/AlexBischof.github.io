---
layout: post
category : pages
tags : [Java, AssertJ, Testing, Design]
---

Recently i stumbled upon [Javin Pauls interview questions article](http://java67.blogspot.de/2013/07/15-java-enum-interview-questions-amswers-for-experienced-programmers.html) 
about enums and although i knew almost everything about it i stopped when i saw the question "Can Enum implement interface in Java". I already had used that
 feature in several projects but more as a marker interface or to provide a more generic way for a module. But this i time recognized the full effect when it comes to 
grouping behavior via inheritance.

Consider the following *Calculator*:
{% highlight bash linenos%}
public interface Calculator {
    default double add(int a, int b){
        return a + b;
    }
}
{% endhighlight %}

Now lets assume that there are several other functional calculators where some of them have the same business logic and therefore can be grouped together. 
Usually (the cleancode way ;) an interface or abstract class is used which can be implemented. In practice this sometimes leads to empty implementation classes to declare
that an implementation exists. But i think it could be improved through the use of enums in the following way:
{% highlight bash linenos%}
public enum BadCalculatorsEnum implements Calculator {
  Bank, InsuranceCompany, Hydra;

  @Override
  public double add(int a, int b) {
    return a + b - 0.001;
  }
}
{% endhighlight %}

To be honest with you this practice has some drawbacks:

 * Since enums can not be extended it can only be used for potential leaf nodes of your inheritance tree
 * In some way it violates the open closed principle when a new group member is added. But i think it is debatable if business logic is not touched.

And last but not least the test...the beauty of Java8, JUnit and AssertJ. 

{% highlight bash linenos%}
@RunWith(Parameterized.class)
public class CalculatorTest {

  @Parameters
  public static Collection<Object[]> data() {
    List<Object[]> data = stream(BadCalculatorsEnum.values())
                .map(c -> new Object[] { c, 0.001 })
                .collect(toList());
    data.add(new Object[] { new Calculator() {}, 0 });
    return data;
  }

  @Parameter
  public Calculator calculator;
  @Parameter(1)
  public double offset;

  @Test
  public void testAdd() throws Exception {
    assertThat(calculator.add(1, 3)).isCloseTo(4, offset(offset));
  }
}
{% endhighlight %}

