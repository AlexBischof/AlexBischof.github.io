---
layout: post
category : pages
tags : [Java, CDI, RapidPM]
disqusid : cdicontextresolver2
---

<p><b>What is the ContextResolver Pattern?</b><br />
It is a pattern described by Sven Ruppert (<a href="http://jaxenter.de/artikel/CDI-entscheide-spaet-entscheide-gut-168301">here</a>) to solve the following problem.<br />
</p>

<b>The Problem</b><br />
A service has several implementations which are provided to clients depending on a specific environment context (for example: test- or developmentcontext) on the service side. The
client does not know about the context and the environment context must be dynamically configurable.<br />
<br />
<b>The Solution</b><br />
Decouple the service creation from the context resolving by introducing<br />
<ul>
    <li>a ContextResolver which determines the current context and returns an annotation literal</li>
    <li>a Service Context Qualifier</li>
    <li>a service producer which uses the servicecontextqualifier</li>
</ul>
With that you can develop very flexible and extendable modules or applications which can be dynamically configured at runtime.<br />
<div>
    <br /></div>
<b>The Evolution</b><br />
The previous version is implemented with CDI extensions which is a little bit harder to understand and needs a&nbsp;<i>javax.enterprise.inject.spi.Extension</i> file. So here
is the improved version which uses only plain CDI-Producers and therefore should be easier to understand.<br />
<br />
The client:<br />
{% highlight java linenos %}
@Inject
@DemoLogicContext
Instance<DemoLogic> demoLogicInst;
{% endhighlight %}
<br />
The producer:
<br />
{% highlight java linenos %}
public class DemoLogicProducer
{
 @Produces
 @DemoLogicContext
 public DemoLogic create(BeanManager beanManager, @Any Instance<ContextResolver> contextResolvers)
 {
  return ManagedBeanCreator.createManagedInstance(beanManager, contextResolvers, DemoLogic.class);
 }
}
{% endhighlight %}
<br />
The ContextResolver:
<br />
{% highlight java linenos %}
public class DemoLogicContextResolver implements ContextResolver
{
 @Inject
 Context context;
 @Override
 public AnnotationLiteral<?> resolveContext(Class<?> targetClass)
 {
  //Determines the context and returns annotionliteral 
  return context.isUseB() ? new MandantB.Literal() : new MandantA.Literal();
 }
}
{% endhighlight %}
<br />
The ManagedBeanCreator:
<br />
{% highlight java linenos %}
public class ManagedBeanCreator
{
 public static <T> T createManagedInstance(BeanManager beanManager, Instance<ContextResolver> contextResolvers,
   Class<? extends T> clazz)
 {
  //FindFirst
  for (ContextResolver contextResolver : contextResolvers)
  {
   AnnotationLiteral<?> annotationLiteral = contextResolver.resolveContext(DemoLogic.class);
   Set<Bean<?>> beans = beanManager.getBeans(clazz, annotationLiteral);
   //
   //Create CDI Managed Bean
   Bean<?> bean = beans.iterator().next();
   CreationalContext<?> ctx = beanManager.createCreationalContext(bean);
   return (T) beanManager.getReference(bean, clazz, ctx);
  }
  return null;
 }
}
{% endhighlight %}


    The sources can be found
    <a href="https://bitbucket.org/abischof/cdicontextresolver2">here</a>.
Have fun coding.