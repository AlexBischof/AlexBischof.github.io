---
layout: post
category : pages
tags : [Java, JUnit, Inheritance, Testing]
disqusid : junit-inheritance
---

This time i want to start with a short quiz. What do you think gets printed if you run the child class test?

{% highlight java linenos %}
public class Child extends Parent{
    @Before
    public void beforeChild(){
        System.out.println("Child before");
    }
    @Test
    public void testChild(){
        System.out.println("Child test");
    }
    @After
    public void afterChild(){
        System.out.println("Child after");
    }
}
{% endhighlight %}
{% highlight java linenos %}
public class Parent {
    @Before
    public void beforeParent(){
        System.out.println("parent before");
    }
    @Test
    public void testParent(){
        System.out.println("parent test");
    }
    @After
    public void afterParent(){
        System.out.println("parent after");
    }
}
{% endhighlight %}

As i mentioned in my first article about JUnit it internally uses reflection to discover all annotations and with that executes in a lifecycle. Knowing that and a little bit of Java reflections it should be rather simple to answer this quiz right. The right answer here is:

{% highlight text linenos %}
parent before
Child before
Child test
Child after
parent after
parent before
Child before
parent test
Child after
parent after
{% endhighlight %}

But the interesting or strange part about that is that the lifecycle methods from the child class are also executed when
 the parent test methods are run. Technically it is total correct but if you are not aware of that you can build up 
 additional complexity to your test classes. There is already a big clean code movement with favour composition over 
 inheritance in order to be more flexible and there should be no except in test code.

