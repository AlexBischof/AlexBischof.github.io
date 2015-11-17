---
layout: post
category : pages
tags : [Java, JavaEE, KumuluzEE, Maven]
---

A couple of days ago I played a little bit with [KumuluzEE](https://ee.kumuluz.com), one of the hot *Duke's Choice Award 2015*
winners, to get a feeling what it is and what it does. So i took one of my JavaEE EJB sample projects and worked myself through
the tutorial and got it running within 30 minutes. From my point of view (whatever this means ;))
there are no big differences to [Wildfly Swarm](wildfly.org/swarm) considering basic *Java EE* development. Sure the *fat-jar*
of the latter is almost twice as big but i think that is related to more configuration possibilities (which i do not know so far, probably something with *subsystems*).

But there is a difference which i find a bit odd and had cost me about 15 minutes because i overlook it in the tutorial ;(. In fact the directory structure differs to standard Java EE/Maven conventions in that way
 that the **webapp** folder has to be located under *scr/main/resources*. The reason for that is probably related to the
 way *com.kumuluz.ee.EeApplication* class is started.
 
{% highlight bash %}
java -cp target/classes:target/dependency/* com.kumuluz.ee.EeApplication
{% endhighlight %}

With the current state *Maven* copies everything under *src/main/resources* also into *target/classes* so that it is automatically in the right directory.

But isn't that a little bit strange? Sure the migration path is rather short. Just copy *webapp* to another folder and it works but what if you want to switch
easily back to *Java EE* containers or like me want to have a profile for *KumuluzEE* or *Wildfly-Swarm* or whatever? (Happy copying ;))

For me it should be encapsulated because a runtime structure should not have influence of the development structure and i think it is always good to hide
internals considering extensibility.

The following part introduces two approaches to let **webapp** at *right* place (*src/main/webapp*):

## 1. Maven Resource Plugin

Just simply copying via *Maven*.

{% highlight xml linenos%}
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.7</version>
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>package</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <outputDirectory>${basedir}/target/classes/webapp</outputDirectory>
                <resources>
                    <resource>
                        <directory>src/main/webapp</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

## 2. KumuluzEE Maven Plugin

With the snippet above and the default *maven-dependency-plugin* your pom.xml has about 40 lines of xml more and i think that this is not going
to increase readability or understandability. On top of that you have to remember internals of *KumuluzEE* which can be change in the future. So 
perfect time to encapsulate all this (and maybe more in the future) in a *Maven plugin*. 

I am no *Maven* expert so please be gentle with the following [KumuluzEE Maven Plugin](https://github.com/AlexBischof/kumuluzee-maven-plugin). The usage
is rather standard.

{% highlight xml linenos%}
<plugin>
    <groupId>de.bischinger</groupId>
    <artifactId>kumuluzee-maven-plugin</artifactId>
    <version>1.0</version>
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
        </execution>
    </executions>
</plugin>
{% endhighlight %}
