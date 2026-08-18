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
Fast alle Softwareentwickler sind sich einig, dass Quellcode lesbar sein soll und für andere Entwickler möglichst lesbar sein soll.

Was allerdings als lesbar und leicht verständlich ist, ist nicht immer so eindeutig. Neben den vereinbarten Styleguides und den eigenen Vorlieben gibt es dann noch andere Kriterien.

Ein Beispiel ist die Validierung von Parametern. Viele generische Bibliotheken beinhalten entsprechende Funktionen. Von den von mir benutzten Bibliotheken dürften zwei der bekanntesten [Google Guava](http://code.google.com/p/guava-libraries/ "Guava: Google Core Libraries for Java 1.6+") und die schon zu den Altmeistern gehörende [apache-commons](http://http://commons.apache.org/ "Apache Commons") Bibliothek sein. Beide beinhalten Methoden zur Validierung von Parametern. Bei Google handelt es sich um die Klasse [Preconditions](http://google-collections.googlecode.com/svn/trunk/javadoc/com/google/common/base/Preconditions.html "Class Preconditions") mit den checkArgument()-Funktionen, bei Apache Commons sind es die Methoden der Klasse [Validate (für Java bis 1.4)](http://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/Validate.html "Class Validate (für Java bis 1.4)") bzw. [Validate (für Java 5+)](http://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/Validate.html "Class Validate (für Java ab 5)"). Egal für welche Variante man sich entscheidet, beide Methoden werfen eine [IllegalArgumentException](http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/IllegalArgumentException.html "IllegalArgumentException").

Jetzt habe ich ein Projekt, in dem beide Klassen (Guava und apache-commons) genutzt werden. Ob das sinnvoll ist, zwei sich soweit überdeckende allgemeine Bibliotheken zu nutzen, wäre einen eigenen Blogpost wert, daher will ich hier nicht weiter darauf eingehen. Allerdings stellt sich dann die Frage, ob man jetzt zu den Precondition-Klassen oder die Validate-Klassen bei der Argumentvalidierung greift:

```java
package de.kaiserpfalzEdv.blog.style;

import static com.google.common.base.Preconditions.checkArgument;

public class ValidationStyle {
 public void checkWithPrecondition(String argument) {
 checkArgument(argument != null && !argument.isEmpty(), "You have to give a not-null and not-empty argument!");

...
 }

...
}
```
```java
package de.kaiserpfalzEdv.blog.style;

import org.apache.commons.lang3.Validate.notEmpty;

public class ValidationStyle {
 public void checkWithValidate(String argument) {
 notEmpty(argument, "You have to give a not-null and not-empty argument!");

...
 }

...
}
```
Auf den ersten Blick sieht die apache-commons-Variante unten eleganter aus. Das ist sie auch, da die Google-Variante immer einen kompletten boolschen Ausdruck erfordert. Allerdings kann man - wenn man sowieso schon Guava und apache-commons eingebunden hat, auch zu einer dritten Variante greifen:

```java
package de.kaiserpfalzEdv.blog.style;

import static com.google.common.base.Preconditions.checkArgument;
import static org.apache.commons.lang3.StringUtils.isNotEmpty;

public class ValidationStyle {
 public void checkWithPreconditionAndStringUtils(String argument) {
 checkArgument(isNotEmpty(argument), "You have to give a not-null and not-empty argument!");

...
 }

...
}
```
In dieser Variante kann mn sofort erkennen, dass ein Argument überprüft wird ("checkArgument") und was denn geprüft wird ("isNotEmpty"). Im Moment ist diese Variante meine Lieblingsvariante. Aber ich bin für Anregungen offen. Die letzte Variante will ich nur noch der Vollständigkeit halber darstellen: alles selbst machen ...

```java
package de.kaiserpfalzEdv.blog.style;

import java.langIllegalArgumentException;

public class ValidationStyle {
 public void checkWithPreconditionAndStringUtils(String argument) {
 if (argument == null || argument.isEmpty) {
 throw new IllegalArgumentException("You have to give a not-null and not-empty argument!");
 }

...
 }

...
}
```
