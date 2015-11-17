---
layout: post
category : pages
tags : [Java, JavaEE, Arquillian, Shrinkwrap, AssertJ, Testing]
---

*Arquillian* is an integration and functional testing framework for JavaEE and i always connect it with the
phrase *Write real world tests*. But what does it mean exactly? What is a real world test?

For me real world test means that the execution environment is the same (or as near as possible) to my production environment and
exactly that is what *Arquillian* provides. 

* You do not need mock frameworks anymore (e.g. Mockito, EasyMock, JMockit)
* You do not have to care about for example the integration of a bean because now you can simply inject it to your test
* With *Shrinkwrap* (bundled into Arquillian) you are able to build an artifact with all your required dependencies so that those are explicit and
makes tests more readable/understandable
* In times of *Microservices* and *Docker* you can also use [Arquillian Cube](https://github.com/arquillian/arquillian-cube) for managed
*Docker* containers. (If you wonder about the name [read here](https://github.com/arquillian/arquillian-cube))

The only drawback i found so far is that it does not work with standard *JUnit* *Parameterized* but this is more related
to *JUnit* since it does not allow more than one runner per test class. Usually this restriction is overcome by implementing 
 the functionality as *JUnit Rule* and fortunately there are two already:
 
 * [http://www.poolik.com/2014/02/how-to-run-parameterized-junit-arquillian-tests](http://www.poolik.com/2014/02/how-to-run-parameterized-junit-arquillian-tests)
 * [https://gist.github.com/aslakknutsen/1358803](https://gist.github.com/aslakknutsen/1358803)
 
I also had some discussions about performance of *Arquillian* tests which are by nature a little bit slower than plain *JUnit* tests. The
standard argument is slow startup time of the containers. But for one thing almost every container (incl. application servers) starts up within
seconds and usually performance on a *Jenkins* is of secondary importance. During development it is recommended to use a *remote container*
so that start up time does not apply anymore.

One important question for me is when to use a framework or tool and when not to use it. From my point of view *Arquillian* can be used in almost
any case except for DAO tests. Sure it works and does not look bad but for me it feels a little bit heavy weight. Especially in comparison
with [Rulz](https://github.com/AdamBien/rulz) of Adam Bien it looks a little bit strange.

Further information about *Arquillian* can be found on the [Arquillian website](http://arquillian.org/).

#### Containers

Since i always forget some of the differences between different container types the following
diagram serves as a helper.

<img src="{{site.baseurl}}/public/images/2015-06-01-arquillianContainers.png"/>

Info:

* Same/Separate JVM: refers to the test runner.
* Isolated/Not Isolated Container: due to the JVM binding (same/separate) a container can have some unexpected effects
 in not isolated containers as you can read 
[here](http://arquillian.org/blog/2012/04/13/the-danger-of-embedded-containers).
* Lifecycle Managed: Means that an container is started as well as shutdown.

The specific container provider can be divided further into the following three groups:

* a fully compliant Java EE application server (e.g. Glassfish, Wildfly)
* a servlet container (e.g. Jetty, Tomcat)
* a standalone bean container (e.g. Weld SE, Spring)

### Usage

In order to run an *Arquillian* test you need to do the following three steps:

* Provide an **arquillian.xml** which declares and configures containers (e.g. arquillian-wildfly-managed)
* Provide **arquillian container dependency** 
    {% highlight java linenos%}
    <plugin>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>2.19</version>
     <configuration>
         <systemPropertyVariables>
                <!-- reference to arquillian.xml container -->
                <arquillian.launch>arquillian-wildfly-managed</arquillian.launch>
            </systemPropertyVariables>
     </configuration>
    </plugin>
    {% endhighlight %}
* Provide **arquillian test**

{% highlight java linenos%}
@Stateless
public class ServiceFacade {
    public String getHello() {return "hello";}
    public String getPrint() {return "print";}
}
{% endhighlight %}

{% highlight java linenos%}
@RunWith(Arquillian.class)
public class ServiceFacadeHelloTest {

    @Inject ServiceFacade serviceFacade;

    @Deployment
    public static WebArchive create() {
           
            //testHello and testPrint are using assertj 
            //therefore the container needs to know it
            File[] libs = Maven.resolver()
                               .loadPomFromFile("pom.xml")
                               .resolve("org.assertj:assertj-core")
                               .withTransitivity().as(File.class);
    
            return ShrinkWrap.create(WebArchive.class, "mytest.war")
                .addAsLibraries(libs)
                .addAsManifestResource(EmptyAsset.INSTANCE, "beans.xml")
                .addClass(ServiceFacade.class);
    }

    @Test
    public void testHello() {
        assertThat(serviceFacade.getHello()).isEqualTo("hello");
    }

    @Test
    public void testPrint() {
        assertThat(serviceFacade.getPrint()).isEqualTo("print");
    }
}
{% endhighlight %}

The snippet above is testing a simple session bean with *AssertJ* via *Arquillian*. The important part for using
*Arquillian* is:
 
1. Declare *Arquillian JUnit* runner with **@RunWith(Arquillian.class)**
2. Provide an archive with a **static** method marked with **@Deployment** which is then deployed to the container. 
In this case ShrinkWrap is used to create a web archive which contains the *AssertJ* library, an empty *beans.xml* and the *ServiceFacade*. 


*Note: JBoss Forge provides already the arquillian-addon.*

The following [repository](https://github.com/AlexBischof/arquillian-remote-test) contains a working
example for three wildfly arquillian containers: embedded, managed and remote.



