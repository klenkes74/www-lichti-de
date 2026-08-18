---
layout: page
status: publish
published: true
title: Konstruktoren, Factory oder Builder? Wie entstehen Objekte?
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 149
wordpress_url: https://www.lichti.de/?p=149
date: '2013-06-09 07:07:12 +0200'
date_gmt: '2013-06-09 07:07:12 +0200'
categories:
- Softwareschnipsel
- Deutsch
tags: []
comments: []
---
Im Netz wurde schon sehr viel geschrieben, wie man Objekte erschaffen kann. Es ist eines der Grundprobleme bei der Softwareentwicklung und wie bei vielen wichtigen Entscheidungen gibt es eine klare Antwort: Es kommt auf die Situation an.

Die einfachste Fassung ist es, den Konstruktor direkt aufzurufen. Das funktioniert immer, wird aber ab einer bestimmten Anzahl von Paramtern leicht unübersichtlich, vor allem wenn viele diese Parameter vom gleichen Typ sind. Das Typenproblem kann aber auch schon bei sehr wenigen Parametern auftreten:

```java
package de.kaiserpfalzEdv.blog.creation;

import static com.google.common.base.Preconditions.checkArgument;
import static org.apache.commons.lang3.StringUtils.isNotBlank;

public class ConstructorDemo {
 ...

public demo() {
 ...
 ValueObject object = new ValueObject("Meier", "Hermann");
 ...
 }
}

class ValueObject {
 private String familyName;
 private String givenName;

public ValueObject(final String familyName, final String givenName) {
 checkArgument(isNotBlank(familyName));
 checkArgument(isNotBlank(givenName));

this.familyName = familyName;
 this.givenName = givenName;
 }

public String getFamilyName() {
 return familyName;
 }

public String getGivenName() {
 return givenName;
 }
}
```
Bei diesem Bespiel könnte es nach einiger Zeit (oder bei einem anderen Entwickler) fraglich sein, ob man zuerst den Nachnamen oder den Vornamen übergeben muss. Noch schlimmer wird es, wenn es vier oder fünf Parameter werden und diese eventuell sogar den gleichen Typ haben - da kann selbst die beste IDE nicht mehr helfen und es wird fast automatisch irgendwann zu Verdrehern kommen.

Das [Builder Pattern](http://de.wikipedia.org/wiki/Erbauer_%28Entwurfsmuster%29 "Builder-Pattern") (deutsch: Erbauer) hilft es, diese Klippe zu umschiffen. Hier hilft eine weitere Klasse, der sogenannte Builder, das Objekt zu erschaffen:

```java
package de.kaiserpfalzEdv.blog.creation;

import java.lang.IllegalStateException;
import static org.apache.commons.lang3.StringUtils.isNotBlank;

public class BuilderDemo {
 ...

public demo() {
 ...
 ValueObject object = new ValueObject.Builder().withFamilyName("Meier").withGivenName("Hermann").build();
 ...
 }
}

class ValueObject {
 private String familyName;
 private String givenName;

private ValueObject() {}

private boolean isValid() {
 return isNotBlank(familyName) && isNotBlank(givenName);
 }

public String getFamilyName() {
 return familyName;
 }

public String getGivenName() {
 return givenName;
 }

public static class Builder() {
 private ValueObject buildObject = new DataClass();

public ValueObject build() {
 if (buildObject.isValid()) {
 return buildObject;
 } else {
 throw new IllegalStateException("Sorry, DataObject can not be created with the given data.");
 }
 }

public Builder withFamilyName(final String name) {
 buildObject.familyName = name;

return this;
 }

public Builder withGivenName(final String name) {
 buildObject.givenName = name;

return this;
 }
 }
}
```
Jetzt können die Paramter nur noch verwechselt werden, wenn der Entwickler nicht weiß, was "familyName" und was "givenName" bedeutet. Aber dann hat man ein ganz anderes Problem. Wie man aber am Code sieht, muss man sich ein paar Gedanken machen: der eigentliche Konstruktor ist nun private, kann also nicht mehr direkt aufgerufen werden.

Das funktioniert, wenn der Builder wie hier eine innere Klasse zum ValueObject darstellt. Bei so kleinen Objekten wie diesem hier geht das auch noch, kann aber unübersichtlich werden.

Bleibt einem so also nicht die Wahl und man muss den Builder als eigene Klasse bauen, dann könnte man das eigentliche Value-Objekt package-local definieren (also ohne "public") und den Builder ins gleiche Package stecken. Der Konstruktor des Value-Objektes müsste demnach dann auch package-lokal sein, was die enge Kapselung leider wieder etwas öffnet. Auch müsste man die Variablen des Value-Objektes package-lokal definieren (oder entsprechende Setter, die wie man oben sieht hier vollständig fehlen).

Im Builder kann man aber auch noch etwas anderes verstecken: Werden oft Objekte mit den gleichen Daten benötigt, so könnten diese in einen Cache gelegt werden und dann diese wieder ausgegeben werden. Da es sich um ValueObjekte handelt, deren Status (hier: Vor- und Nachname) sich nicht mehr ändern kann, kann auch die Software in Wirklichkeit mit einem einzelnen Objekt arbeiten und braucht nicht immer ein neues Objekt. Es kann also dadurch optimiert werden.

Was mache ich aber, wenn die die Möglichkeit des Wiederverwendens eines Objekts nutzen will, aber ein Builder mit Kanonen auf Spatzen geschossen wäre (zum Beispiel, wenn ich nur einen Parameter habe)?

Dann kann ich eine [Factory](http://en.wikipedia.org/wiki/Factory_%28software_concept%29 "Factory") nutzen.

```java
package de.kaiserpfalzEdv.blog.creation;

import static com.google.common.base.Preconditions.checkArgument;
import static org.apache.commons.lang3.StringUtils.isNotBlank;

public class ConstructorDemo {
 ...

public demo() {
 ...
 ValueObject object = ValueObject.Factory.getInstance("Meier", "Hermann");
 ...
 }
}

class ValueObject {
 private String familyName;
 private String givenName;

private ValueObject(final String familyName, final String givenName) {
 this.familyName = familyName;
 this.givenName = givenName;
 }

public String getFamilyName() {
 return familyName;
 }

public String getGivenName() {
 return givenName;
 }

public static class Factory {
 public static ValueObject getInstance(final String familyName, final String givenName) {
 checkArgument(isNotBlank(familyName));
 checkArgument(isNotBlank(givenName));

return new ValueObject(familyName, givenName);
 }
 }
}
```
Die Factory unterscheidet sich von der [Factory-Methode](http://en.wikipedia.org/wiki/Factory_method_pattern "Factory-Method") dadurch, dass sie eine eigene Klasse ist, die eine oder mehrere Factory-Methoden in sich vereint, während die Factory-Methode eine statische Methode in der eigentlichen Klasse darstellt:

```java
package de.kaiserpfalzEdv.blog.creation;

import static com.google.common.base.Preconditions.checkArgument;
import static org.apache.commons.lang3.StringUtils.isNotBlank;

public class ConstructorDemo {
 ...

public demo() {
 ...
 ValueObject object = ValueObject.getInstance("Meier", "Hermann");
 ...
 }
}

class ValueObject {
 private String familyName;
 private String givenName;

private ValueObject(final String familyName, final String givenName) {
 this.familyName = familyName;
 this.givenName = givenName;
 }

public static ValueObject getInstance(final String familyName, final String givenName) {
 checkArgument(isNotBlank(familyName));
 checkArgument(isNotBlank(givenName));

return new ValueObject(familyName, givenName);
 }

public String getFamilyName() {
 return familyName;
 }

public String getGivenName() {
 return givenName;
 }
}
```
Damit haben wir nun alle vier Möglichkeiten gesehen. Welche passt am besten? Und die Antwort hatten wir schon zu Anfang: Es kommt auf die Situation an. Eventuell kann die folgende Kurzliste helfen ...

1. Das Objekt hat nur wenige (1-3) Parameter, die auch nicht leicht zu verwechseln sind. Außerdem soll es jedesmal neu geschaffen werden. Ein typischer Fall für den Konstruktor-Aufruf per "new Object(...)".
2. Das Objekt hat nur wenige (1-3) Parameter, die auch nicht leicht zu verwechseln sind. Es soll eventuell gecacht werden. Ein typischer Fall für die Factory-Method innerhalb des Objekts.
3. Das Objekt hat nur wenige (1-3) Parameter, die auch nicht leicht zu wechseln sind. Es soll eventuell gecacht werden oder gar eine Subklasse generiert werden (abhängig von den Parametern). Ein typischer Fall für die Factory.
4. Das Objekt hat viele Paramter oder wenige Paramter, die leicht zu verwechseln sind. Hier kann der Builder echt helfen ...
