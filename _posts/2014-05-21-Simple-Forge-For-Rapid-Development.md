---
layout: post
category : pages
tags : [Java, JBoss, Forge, JavaEE]
disqusid : simple-forge
---
<p>
This article covers the simple usage of Forge to build up an application (or at least an almost complete stack for a prototype).
So what is Forge? It is a command line tool to create and configure Java projects. You can setup different modules 
(for example cdi or jpa) and Forge generates all resources needed. This can be anything from java classes to deployment
 descriptors right up to java test classes or JSF files. On top of that this behavior is incremental, so that you can 
 use it at any time you want.
</p>

The setup of the modules is done by Convention over Configuration so that everything is quite simple and comprehensible. The Command line also supports tab-completion and makes the usage very fluent.<br />
<br />
You may ask why not Forge 2 (it is already in version 2.5.0). It is faster and a little bit easier but it lacks in one point: the Arquillian module is currently not migrated but it is planed.<br />
<br />
<b>Example</b><br />
At first we have to tell Forge to use defaults. Then we create a project with name forgetest and the topLevelPackage de.bischinger. With this a Maven War project is created.<br />
<pre style="background-color: #f8f8f8; border-bottom-left-radius: 3px; border-bottom-right-radius: 3px; border-top-left-radius: 3px; border-top-right-radius: 3px; border: 1px solid rgb(221, 221, 221); box-sizing: border-box; color: #333333; font-family: Consolas, 'Liberation Mono', Courier, monospace; font-size: 13px; line-height: 19px; margin-bottom: 15px; margin-top: 15px; overflow: auto; padding: 6px 10px; word-wrap: normal;">[no project] Development $ set ACCEPT_DEFAULTS true;
[no project] Development $ new-project --named forgetest --topLevelPackage de.bischinger;</pre>
After this you can see that the shell changes from "no project" to your project name "forge test".<br />
Now we want to add persistence namely Hibernate and WildFly (creates a persistence.xml and adds dependency to pom.xml)<br />
<pre style="background-color: #f8f8f8; border-bottom-left-radius: 3px; border-bottom-right-radius: 3px; border-top-left-radius: 3px; border-top-right-radius: 3px; border: 1px solid rgb(221, 221, 221); box-sizing: border-box; color: #333333; font-family: Consolas, 'Liberation Mono', Courier, monospace; font-size: 13px; line-height: 19px; margin-bottom: 15px; margin-top: 15px; overflow: auto; padding: 6px 10px; word-wrap: normal;">[forgetest] forgetest $ persistence setup --provider HIBERNATE --container WILDFLY;
</pre>
Create a JPA entity customer with a required field name. For this we need also to setup JPA validation.<br />
<pre style="background-color: #f8f8f8; border-bottom-left-radius: 3px; border-bottom-right-radius: 3px; border-top-left-radius: 3px; border-top-right-radius: 3px; border: 1px solid rgb(221, 221, 221); box-sizing: border-box; color: #333333; font-family: Consolas, 'Liberation Mono', Courier, monospace; font-size: 13px; line-height: 19px; margin-bottom: 15px; margin-top: 15px; overflow: auto; padding: 6px 10px; word-wrap: normal;">[forgetest] forgetest $ validation setup --provider HIBERNATE_VALIDATOR;
[forgetest] forgetest $ entity --named Customer --package de.bischinger.model;
[forgetest] Customer.java $ field string --named name;
[forgetest] Customer.java $ constraint NotNull --onProperty name;
</pre>
The interesting part is once again that after the creation the shell switches to the created context so that you can easily create the fields. You can also list the context with <i>ls</i>.<br />
<br />
Now we want to build a simple JSF UI for the customer entity. At first we need to setup Scaffold which generates our UI.<br />
<pre style="background-color: #f8f8f8; border-bottom-left-radius: 3px; border-bottom-right-radius: 3px; border-top-left-radius: 3px; border-top-right-radius: 3px; border: 1px solid rgb(221, 221, 221); box-sizing: border-box; color: #333333; font-family: Consolas, 'Liberation Mono', Courier, monospace; font-size: 13px; line-height: 19px; margin-bottom: 15px; margin-top: 15px; overflow: auto; padding: 6px 10px; word-wrap: normal;">[forgetest] Customer.java $ scaffold setup;
[forgetest] Customer.java $ scaffold from-entity de.bischinger.model.Customer.java;
</pre>
After this we can build our application with <i>build</i>&nbsp;and after deploying (which can also be done with Forge but this would be another topic) we can see the following JSF-Page. On the left side is our link to the customers where we can already create, search and delete customers. It also has a paging feature.<br />
<div class="separator" style="clear: both; text-align: center;">
    <a href="http://4.bp.blogspot.com/-aD-MwxP_yr8/U3xRxzDgRUI/AAAAAAAAAC0/CeFU1hJQRiY/s1600/Bildschirmfoto+2014-05-21+um+09.10.41.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><high border="0" src="http://4.bp.blogspot.com/-aD-MwxP_yr8/U3xRxzDgRUI/AAAAAAAAAC0/CeFU1hJQRiY/s1600/Bildschirmfoto+2014-05-21+um+09.10.41.png" height="177" width="400" /></a></div>
<div>
    Ok - the GUI still must be customized but this is again another topic. The point is that i have spent so much time in the past to reach a state like this - now it is possible within just 2 minutes.</div>
<br />
But we are not finished yet - what about tests? For this we can install the Arquillian plugin into our project. We setup Arquillian to use a WildFly-Managed-Container with JUnit after which we can create our tests.<br />
<pre style="background-color: #f8f8f8; border-bottom-left-radius: 3px; border-bottom-right-radius: 3px; border-top-left-radius: 3px; border-top-right-radius: 3px; border: 1px solid rgb(221, 221, 221); box-sizing: border-box; color: #333333; font-family: Consolas, 'Liberation Mono', Courier, monospace; font-size: 13px; line-height: 19px; margin-bottom: 15px; margin-top: 15px; overflow: auto; padding: 6px 10px; word-wrap: normal;">[forgetest] Customer.java $ forge install-plugin arquillian;
[forgetest] Customer.java $ arquillian setup --containerName WILDFLY_MANAGED --testFramework junit;
[forgetest] Customer.java $ arquillian create-test --class de.bischinger.model.Customer.java;</pre>
Done.<br />
<br />
<b>Conclusion</b><br />
<b><br /></b>
From my point of view this tool is worth a lot. I can integrate and configure almost any standard technology (JPA, Validation...) i want in no time. This way i can really concentrate on the business. Forge is nicely integrated in the JBoss Developer Studio, it can also be used on any shell and it is easy to understand. The generated sources can also be studied to familiarize with unknown technologies.<br />
<br />