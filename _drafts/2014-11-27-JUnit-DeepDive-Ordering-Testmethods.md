---
layout: post
category : pages
tags : [Java, JUnit, Testing]
disqusid : junit-ordering
---
One of the more controversial topics in the junit world is whether test methods can or should be executed in order or not.
So it should not be surprising that ordering of test methods is a 
relatively new feature in the JUnit framework which is since 4.11.

Currently there are three different algorithms:
* DEFAULT: Sorts the test methods in deterministic order but this is not predictable. Internally the hashcode of the method name string is used. This also means that by default all test methods are ordered.
* JVM: No sorting which means that the order of the JVM is used. With this typ you have to be aware that the order can change on different JVMs.
* NAME_ASCENDING: Sorts the test methods by method name in lexicographic order.

The Usage is fairly easy, just annotate your test class with @FixMethodOrder annotation like this

{% highlight java linenos %}
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class SortingTestCase {
    @Test public void test2(){
        System.out.println("hello2");
    }
    @Test public void test(){
        System.out.println("hello");
    }
}
{% endhighlight %}
 and all your test methods of the class are executed in order.<br/>

Internally the ordering functionality is attached to the internal class <i>TestClass</i> which is called in the constructor of the parent runner. So if you build up your own runner and inherit from parent runner you are safe and can use the build in ordering. Interesting for me was that no mechanism for lifecycle methods or fields is provided but i think there is currently no need for it. <br/>
Considering extending the existing three algorithms there is currently no way besides reimplement the whole code and use it in a custom runner. 
But also i can not imagine any case where this should be needed.

Also be aware that if you have jdk6 to jdk7_b127 there is a bug out there which can break your assumptions. You can read more about it here. TODO Link 

So, now comes the interesting part. When should i use test method ordered execution? 
In general tests should not rely on any order and should be isolated and independent which is also stated in the JUnit <a href="http://junit.org/faq.html>FAQ</a>.
The problem is that in some cases where state machines are involved and setting the state is. 

The more famous discussions on stack overflow can be found 
<a href="http://stackoverflow.com/questions/9528581/specifying-order-of-execution-in-junit-test-case“>here</a> and 
<a href="http://stackoverflow.com/questions/3693626/how-to-run-test-methods-in-specific-order-in-junit4>here</a> and are quiet interesting. 
