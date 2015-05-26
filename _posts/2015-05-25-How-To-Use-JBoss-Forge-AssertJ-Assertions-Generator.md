---
layout: post
category : pages
tags : [Java, AssertJ, JBoss Forge, Testing]
---

This time i want to present the JBoss-Forge AssertJ Addon which is brand new and basically does the maven integration of
the [AssertJ Assertions Generator](http://joel-costigliola.github.io/assertj/assertj-assertions-generator.html)

If you are not familiar with either [JBoss Forge](http://forge.jboss.org/) or [AssertJ](http://assertj.org) you really 
should have a look on that because it will speed up your development.

### Installation

Before you can use this addon you have to install it first. 

{% highlight bash linenos%}
forge -i org.assertj.forge:assertj-forge-addon
{% endhighlight %}

### Minimal Setup

The minimal setup for *AssertJ Assertions Generator* **requires** either a **package** or a **class name** for which the generator should
be used. To do so you need to create a project first.

{% highlight bash linenos%}
[github]$ project-new --named petstore
***SUCCESS*** Project named 'petstore' has been created.
[petstore]$ assertj-assertions-generator-setup --packages org.petshop.model
***SUCCESS*** Command 'assertj-setup' successfully executed!
{% endhighlight %}

The *assertj-assertions-generator-setup* command basically:

 * Sets 2 version properties
 * Adds the *AssertJ* dependency
 * Adds the *assertj-assertions-generator-maven-plugin*

The resulting *pom.xml* looks like this:

{% highlight bash linenos%}
[petstore]$ cat pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.petstore</groupId>
  <artifactId>petstore</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>war</packaging>
  <properties>
    <version.assertj>2.0.0</version.assertj>
    <version.assertj-generator>1.6.0</version.assertj-generator>
    <maven.compiler.source>1.7</maven.compiler.source>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.assertj</groupId>
      <artifactId>assertj-core</artifactId>
      <version>${version.assertj}</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>petstore</finalName>
    <plugins>
      <plugin>
        <artifactId>maven-war-plugin</artifactId>
        <version>2.4</version>
        <configuration>
          <failOnMissingWebXml>false</failOnMissingWebXml>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-assertions-generator-maven-plugin</artifactId>
        <version>${version.assertj-generator}</version>
        <executions>
          <execution>
            <goals>
              <goal>generate-assertions</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <packages>
            <param>org.petshop.model</param>
          </packages>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
{% endhighlight %}

The following bash snippet shows all parameters of that command which are described either in the man page of *assertj-assertions-generator-setup*
or [AssertJ Assertions Generator](http://joel-costigliola.github.io/assertj/assertj-assertions-generator-maven-plugin.html#configuration). 

{% highlight bash linenos%}
[petstore]$ assertj-assertions-generator-setup 
--version                         --excludes                        --notGenerateAssertions
--notGenerateJUnitSoftAssertions  --packages                        --includes
--notGenerateBddAssertions        --entryPointClassPackage          --classes
--notHierarchical                 --notGenerateSoftAssertions       --targetDir  
{% endhighlight %}

AFAIK there is now faster way to use your domain model assertions in your tests then this.

The code for that can be found on my github repositoy: [https://github.com/AlexBischof/assertj-forge-addon](https://github.com/AlexBischof/assertj-forge-addon)





