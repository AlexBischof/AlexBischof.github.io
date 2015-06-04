---
layout: post
category : pages
tags : [Java, Java9, Migration]
---

Yesterday i had looked what happens if i build [AssertJ](http://www.assertj.org) with the current *Java9* (1.9.0-ea-b66) release
and stumbled upon some things which i had not expected. To be true here i had expected that everything works fine. Mainly because
*AssertJ* has almost no dependencies and does not use some fancy stuff like *Unsafe*. Maybe i was too optimistic. ;)
 
### Problem 1: Compiler error with bounded generics with arrays/varargs

The first problem involves a specific combination of 

  * arrays/varargs
  * bounded generics 
  * inferred types
  
and appears as a **compiler error**. Consider the following example which works fine with the latest Java8 (1.8.0_45-b14).

{% highlight java linenos%}
public class Example {
    @SafeVarargs
    public static <T> A<T> create(A<? extends T>... a) {
        return allOf(a);        //Compile error
    }

    @SafeVarargs
    public static <T> A<T> allOf(A<? extends T>... a) {
        return null;
    }

    class A<T> {}
}
{% endhighlight %}

{% highlight bash linenos%}
de/bischinger/jdk9test/Example.java:11: error: method allOf in class
 Example cannot be applied to given types;
        return allOf(a);
               ^
  required: Example.A<? extends T#1>[]
  found: Example.A<? extends T#2>[]
  reason: cannot infer type-variable(s) T#1
    (varargs mismatch; Example.A<? extends T#2>[] cannot be converted
     to Example.A<? extends T#1>)
  where T#1,T#2 are type-variables:
    T#1 extends Object declared in method <T#1>allOf(Example.A<? extends T#1>...)
    T#2 extends Object declared in method <T#2>create(Example.A<? extends T#2>...)
1 error
{% endhighlight %}

As the error message points out (*with -Xdiags:verbose*) *Java9* is more strict when it comes to such constellations. I am
not sure why this change was really necessary but it probably came with [JEP-213](http://openjdk.java.net/jeps/213).

#### Solution or Workaround?

The only workaround to that problem i was able to found looks as follows

{% highlight java linenos%}
    public static <T> A<T> create(A<? extends T>... a) {
            return Example.<T>allOf(a);
    }
{% endhighlight %}

and looks ugly. The thing which makes it more ugly is that this cannot be done with an automatic refactoring via an IDE. ;(

### Problem 2: XML PrettyPrint

The other thing i had found is that the behavior of XML pretty print does not work anymore. At first
this does not sound like a problem but if your software is using this functionality and makes use of *equals* (which testing frameworks or tests sometimes do)
then you probably have a problem.

At the moment i have not found a solution/workaround for it. So if you have any information about that please let me know.

{% highlight java linenos%}
public class PrettyPrint {
    public static void main(String[] args) throws Exception {
        String xml = "<rss version=\"2.0\"><channel>  <title>Java Tutorials and Examples 1</title>  <language>en-us</language></channel></rss>";
        Writer stringWriter = createPrettyPrint(xml);
        System.out.println(stringWriter.toString());
    }

    private static Writer createPrettyPrint(String xml) throws Exception {
        DOMImplementationRegistry registry = DOMImplementationRegistry.newInstance();
        DOMImplementationLS domImplementation = (DOMImplementationLS) registry.getDOMImplementation("LS");
        Writer stringWriter = new StringWriter();
        LSOutput formattedOutput = domImplementation.createLSOutput();
        formattedOutput.setCharacterStream(stringWriter);
        LSSerializer domSerializer = domImplementation.createLSSerializer();
        domSerializer.getDomConfig().setParameter("format-pretty-print", true);
        // Set this to true if the declaration is needed to be in the output.
        domSerializer.getDomConfig().setParameter("xml-declaration", true);
        domSerializer.write(toXmlDocument(xml), formattedOutput);
        return stringWriter;
    }

    private static Document toXmlDocument(String xmlString) throws Exception {
        InputSource xmlInputSource = new InputSource(new StringReader(xmlString));
        DocumentBuilder xmlDocumentBuilder = DocumentBuilderFactory.newInstance().newDocumentBuilder();
        return xmlDocumentBuilder.parse(xmlInputSource);
    }
}
{% endhighlight %}

Java9
{% highlight java linenos%}
<?xml version="1.0" encoding="UTF-8"?><rss version="2.0">
    <channel>  <title>Java Tutorials and Examples 1</title>  <language>en-us</language>
    </channel>
</rss>
{% endhighlight %}

Java8
{% highlight java linenos%}
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
    <channel>
        <title>Java Tutorials and Examples 1</title>
        <language>en-us</language>
    </channel>
</rss>
{% endhighlight %}

