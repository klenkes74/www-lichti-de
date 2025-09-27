---
layout: page
status: publish
published: true
title: 'Lesbarer Quellcode - Heute: Wie validiere ich Methodenparameter?'
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 130
wordpress_url: https://www.lichti.de/?p=130
date: '2013-03-31 11:11:30 +0200'
date_gmt: '2013-03-31 11:11:30 +0200'
categories:
- Softwareschnipsel
- Deutsch
tags: []
comments: []
---
<p>Fast alle Softwareentwickler sind sich einig, dass Quellcode lesbar sein soll und für andere Entwickler möglichst lesbar sein soll.</p>
<p>Was allerdings als lesbar und leicht verständlich ist, ist nicht immer so eindeutig. Neben den vereinbarten Styleguides und den eigenen Vorlieben gibt es dann noch andere Kriterien.</p>
<p><!--more--></p>
<p>Ein Beispiel ist die Validierung von Parametern. Viele generische Bibliotheken beinhalten entsprechende Funktionen. Von den von mir benutzten Bibliotheken dürften zwei der bekanntesten <a title="Guava: Google Core Libraries for Java 1.6+" href="http://code.google.com/p/guava-libraries/" target="_blank" rel="noopener noreferrer">Google Guava</a> und die schon zu den Altmeistern gehörende <a title="Apache Commons" href="http://http://commons.apache.org/" target="_blank" rel="noopener noreferrer">apache-commons</a> Bibliothek sein. Beide beinhalten Methoden zur Validierung von Parametern. Bei Google handelt es sich um die Klasse <a title="Class Preconditions" href="http://google-collections.googlecode.com/svn/trunk/javadoc/com/google/common/base/Preconditions.html" target="_blank" rel="noopener noreferrer">Preconditions</a> mit den checkArgument()-Funktionen, bei Apache Commons sind es die Methoden der Klasse <a title="Class Validate (für Java bis 1.4)" href="http://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/Validate.html" target="_blank" rel="noopener noreferrer">Validate (für Java bis 1.4)</a> bzw. <a title="Class Validate (für Java ab 5)" href="http://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/Validate.html" target="_blank" rel="noopener noreferrer">Validate (für Java 5+)</a>. Egal für welche Variante man sich entscheidet, beide Methoden werfen eine <a title="IllegalArgumentException" href="http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/IllegalArgumentException.html" target="_blank" rel="noopener noreferrer">IllegalArgumentException</a>.</p>
<p>Jetzt habe ich ein Projekt, in dem beide Klassen (Guava und apache-commons) genutzt werden. Ob das sinnvoll ist, zwei sich soweit überdeckende allgemeine Bibliotheken zu nutzen, wäre einen eigenen Blogpost wert, daher will ich hier nicht weiter darauf eingehen. Allerdings stellt sich dann die Frage, ob man jetzt zu den Precondition-Klassen oder die Validate-Klassen bei der Argumentvalidierung greift:</p>
<p>[java autolinks="false" collapse="false" firstline="1" gutter="true" highlight="3,7" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="4" toolbar="true"]<br />
package de.kaiserpfalzEdv.blog.style;</p>
<p>import static com.google.common.base.Preconditions.checkArgument;</p>
<p>public class ValidationStyle {<br />
    public void checkWithPrecondition(String argument) {<br />
        checkArgument(argument != null &amp;&amp; !argument.isEmpty(), &quot;You have to give a not-null and not-empty argument!&quot;);</p>
<p>        ...<br />
    }</p>
<p>    ...<br />
}<br />
[/java]<br />
[java autolinks="false" collapse="false" firstline="1" gutter="true" highlight="3,7" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="4" toolbar="true"]<br />
package de.kaiserpfalzEdv.blog.style;</p>
<p>import org.apache.commons.lang3.Validate.notEmpty;</p>
<p>public class ValidationStyle {<br />
    public void checkWithValidate(String argument) {<br />
        notEmpty(argument, &quot;You have to give a not-null and not-empty argument!&quot;);</p>
<p>        ...<br />
    }</p>
<p>    ...<br />
}<br />
[/java]</p>
<p>Auf den ersten Blick sieht die apache-commons-Variante unten eleganter aus. Das ist sie auch, da die Google-Variante immer einen kompletten boolschen Ausdruck erfordert. Allerdings kann man - wenn man sowieso schon Guava und apache-commons eingebunden hat, auch zu einer dritten Variante greifen:</p>
<p>[java autolinks="false" collapse="false" firstline="1" gutter="true" highlight="3-4,8" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="4" toolbar="true"]<br />
package de.kaiserpfalzEdv.blog.style;</p>
<p>import static com.google.common.base.Preconditions.checkArgument;<br />
import static org.apache.commons.lang3.StringUtils.isNotEmpty;</p>
<p>public class ValidationStyle {<br />
    public void checkWithPreconditionAndStringUtils(String argument) {<br />
        checkArgument(isNotEmpty(argument), &quot;You have to give a not-null and not-empty argument!&quot;);</p>
<p>        ...<br />
    }</p>
<p>    ...<br />
}<br />
[/java]</p>
<p>In dieser Variante kann mn sofort erkennen, dass ein Argument überprüft wird ("checkArgument") und was denn geprüft wird ("isNotEmpty"). Im Moment ist diese Variante meine Lieblingsvariante. Aber ich bin für Anregungen offen. Die letzte Variante will ich nur noch der Vollständigkeit halber darstellen: alles selbst machen ...</p>
<p>[java autolinks="false" collapse="false" firstline="1" gutter="true" highlight="3,7-9" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="4" toolbar="true"]<br />
package de.kaiserpfalzEdv.blog.style;</p>
<p>import java.langIllegalArgumentException;</p>
<p>public class ValidationStyle {<br />
    public void checkWithPreconditionAndStringUtils(String argument) {<br />
        if (argument == null || argument.isEmpty) {<br />
            throw new IllegalArgumentException(&quot;You have to give a not-null and not-empty argument!&quot;);<br />
        }</p>
<p>        ...<br />
    }</p>
<p>    ...<br />
}<br />
[/java]</p>
