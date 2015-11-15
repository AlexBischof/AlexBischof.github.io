---
layout: post
category : pages
tags : [Java, JUnit, Testing]
---

### Follow up

As a small follow up on rules i want to provide two usecases which i am using quite often in my projects: 

1. JPA/JAXRS Rules:<br/>
In almost every of my projects where i typically work more as a backend java engineer i have to deal with JPA/Hibernate or JAXRS endpoints and therefore 
have to write tests for those services. And with that i mean more simple DAO or endpoint tests otherwise i use [Arquillian](http://arquillian.org/) but that
is another topic. In order to not reinventing the wheel i use the rules from Adam Bien which can be found [here](https://github.com/AdamBien/rulz).
  
2. Overcome JUnit restrictions of runners <br/>
The current release of JUnit (version 4.12) has the restriction that a test class can only have one runner. Therefore it is not possible to combine 
functionalities of multiple runners which in some case can really be useful. But with the concept of rules it is possible if the functionality 
 of one runner is extracted into a rule. Fortunately for us there are plenty of rules available already. Here are two typical problems: 

* Spring and Mockito: [https://www.holisticon.de/2011/03/unit-tests-im-spring-context-mit-junit-und-mockito](https://www.holisticon.de/2011/03/unit-tests-im-spring-context-mit-junit-und-mockito) 
* Parameterized Rule: [http://blog.schauderhaft.de/2012/12/16/writing-parameterized-tests-with-junit-rules](http://blog.schauderhaft.de/2012/12/16/writing-parameterized-tests-with-junit-rules) 

### Quiz

This short review quiz is related to the previous article [DeepDive Part3 Rules](http://alexbischof.github.io/JUnit-DeepDive-Part3-Rules/).

A. Which of the following choices do you have to apply to get non class rules running?
  
  1. add annotation *@Rule*
  2. make rule member public
  3. add *@RunWith(RuleRunner.class)*
  4. initialize member
  
B. Which of the following statements are true?
 
  1. @Before/@After lifecycle hooks are always execute before rules.
  2. @Before/@After lifecycle hooks are always execute after rules.
 
C. What happens in the following code snippet?
 
{% highlight bash linenos%}
public class RuleTest {
    @Rule public LoggingRule rule = new LoggingRule();

    @Test public void test() {
        System.out.println("test");
    }

    @AfterClass public static void afterClass() {
        System.out.println("afterclass");
    }
}

class LoggingRule implements TestRule {
    @Override
    public Statement apply(Statement base, Description description) {
        return new Statement() {
            @Override
            public void evaluate() throws Throwable {
                System.out.println("LoggingRule");
                base.evaluate();
            }
        };
    }
}
{% endhighlight %}
 
  1. Prints LoggingRule, test, afterclass.
  2. Prints test, afterclass.
  3. throws NullPointerException.
  
D. Okay, the previous one was easy. What happens if you change line 2 into
{% highlight bash linenos%}
@Rule public LoggingRule rule;
{% endhighlight %}

  1. Prints LoggingRule, test, afterclass.
  2. Prints test, afterclass.
  3. throws NullPointerException.

The solutions can be found here in white color: <font color="white">A124; B2; C1; D2 </font>
 
 