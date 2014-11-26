---
layout: post
category : pages
tags : [Java, Wildfly, JBoss, Security, PBKDF2]
disqusid : wildflyjbosspasswordsecurity
---

<div>
    If you are familiar with Wildfly/JBoss you might now that you can easily configure your system to use JAAS database authentication  for the login process of your customer. Because it is considered bad practice to store passwords in clear text a hash-algorithm (e.g. SHA256, MD5) can be specified to encrypt them. Further informations can be found under&nbsp;<a href="https://docs.jboss.org/author/display/WFLY8/Security+subsystem+configuration">here</a>.</div>
<div>
    <br /></div>
<div>
    The problem with that approach is that your passwords will be vulnerable to dictionary and rainbowtable attacks (refresh your knowlegde&nbsp;<a href="http://netsecurity.about.com/od/hackertools/a/Rainbow-Tables.htm">here</a>).</div>
<div>
    One way to face this problem is to use salted hashed passwords but the standard DatabaseServerLoginModule does not support that. But you can use <a href="http://www.rtner.de/software/PBKDF2.html">this</a>&nbsp;extension of the DatabaseServerLoginModule.</div>
<div>
    The configuration is fairly easy:</div>
<div>
    <br /></div>
<h3>
    Wildfly/JBoss Konfiguration</h3>
<div>
    <b>1</b>. Create module</div>
<div>
    Create the new module de.rtner.security.main and configure the module.xml like this</div>
<div>
{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<module xmlns="urn:jboss:module:1.1" name="de.rtner.security">
    <resources>
        <resource-root path="PBKDF2-1.0.4.jar"/>
        <resource-root path="SaltedDatabaseLoginModule-1.0.4.jar"/>
    </resources>
   <dependencies>
        <module name="javax.api"/>
        <module name="javax.servlet.api"/>
        <module name="org.jboss.as.web-common"/>
        <module name="org.jboss.logging"/>
        <module name="org.picketbox"/>
        <module name="sun.jdk"/>
    </dependencies>
</module>
{% endhighlight %}
    Both declared jars must also be present in that folder.</div>
<div>
    <br /></div>
<div>
    <b>2</b>. Update module.xml of picketbox.</div>
<div>
    Every custom login module has to be declared as a dependencies in the module.xml of picketbox. So just add this line:<br />
    <pre><module name="de.rtner.security"/></pre>
</div>
<div>
    <b>3</b>. Update standalone.xml</div>
{% highlight xml linenos %}
<login-module code="de.rtner.security.auth.spi.SaltedDatabaseServerLoginModule" flag="required" module="de.rtner.security">
 <module-option name="dsJndiName" value="java:jboss/datasources/ExampleDS"/>
 <module-option name="principalsQuery" value="select password from User where login=?"/>
 <module-option name="rolesQuery" value="select role, 'Roles' from UserRole where login=?"/>
 <module-option name="hmacAlgorithm" value="HMacSHA256"/>
 <module-option name="formatter" value="de.rtner.security.auth.spi.PBKDF2HexFormatter"/>
 <module-option name="engine" value="de.rtner.security.auth.spi.PBKDF2Engine"/>
 <module-option name="engine-parameters" value="de.rtner.security.auth.spi.PBKDF2Parameters"/>
</login-module>
{% endhighlight %}
The password encryption can be done with the following function.
<br />
<div>
{% highlight java linenos %}private String createPbKdF2Passwort(String password) throws NoSuchAlgorithmException
{
 PBKDF2Formatter formatter = new PBKDF2HexFormatter();
 SecureRandom sr = SecureRandom.getInstance("SHA1PRNG");
 byte[] salt = new byte[8];
 sr.nextBytes(salt);
 int iterations = 1000;
 PBKDF2Parameters p = new PBKDF2Parameters("HmacSHA256", "ISO-8859-1", salt, iterations);
 PBKDF2Engine e = new PBKDF2Engine(p);
 p.setDerivedKey(e.deriveKey(password));
 return formatter.toString(p);
}{% endhighlight %}