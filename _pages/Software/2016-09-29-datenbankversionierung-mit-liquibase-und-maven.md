---
layout: page
status: publish
published: true
title: Datenbankversionierung mit Liquibase und Maven
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 255
wordpress_url: https://www.lichti.de/?p=255
date: '2016-09-29 05:46:12 +0200'
date_gmt: '2016-09-29 03:46:12 +0200'
categories:
- Softwareschnipsel
- Deutsch
tags:
- build
- ci
- configurationmanagement
- liquibase
- maven
comments: []
---
SQL-Datenbanken gehören trotz der großen Aufmerksamkeit für NOSQL zum Brot-und-Butter-Handwerkzeug eines Softwareentwicklers. Und zumindest im Unternehmensumfeld wird des wohl noch lange so bleiben, denn dort setzen sich neue Konzepte nur langsam durch. Doch schon während der Softwareentwicklung stößt man im Team auf das Problem, dass Änderungen an den Datenstrukturen verwaltet werden müssen. Die Kollegen brauchen bei Softwareänderungen auch geänderte Testdatenbanken, man will auch nachhalten, welcher Commit welche Datenbankänderung nach sich zog.

Hier kommen dann Datenbankversionierungstools wie [Flyway](ttps://flywaydb.org) oder eben [Liquibase](http://www.liquibase.org/) ins Spiel. Flyway habe ich uns vor langer Zeit nur kurz angeschaut, aber eine persönliche Präferenz des für das damalige Projekt verantwortlichen Softwarearchitekten für XML-basierte Tools hat mich dann zu Liquibase geführt. Wie es in ein Mavenprojekt integriert werden kann, damit Integrationstests immer auf eine aktuelle Datenbank zugreifen können, will ich im folgenden beschreiben.

Ich werde hier nicht näher darauf eingehen, wie man zum Beispiel einen [H2-Server in einen Maven-Integrationstest](https://www.lichti.de/2016/09/29/h2-datenbankserver-mit-maven/) einbauen kann. Ich gehe von einem laufenden Server aus, um dann die Datenbankstruktur validieren und updaten zu können.

## Die Verwaltung der Datenbankänderungen

Man braucht natürlich die Dateien, die die Datenbank und ihre Änderungen beschreiben. Ich habe mir Angewöhnt, nicht erst beim 1. Update in der Produktion mit Liquibase zu arbeiten sondern bereits in der Entwicklung damit zu starten. Im Team kann man so auch leicht Strukturänderungen an den Datenbanken übernehmen und damit kommunizieren. Außerdem werden sie so im Sourcecodemanagement-Tool verwaltet und können (und sollten!) Bestandteil des Code-Review im Team werden. Ich fange meist mit einer zentralen Changelog-Datei an, die dann weitere Dateien einbindet. Je nach Komplexität werden in den eingebundenen Dateien direkt Datenbankänderungen konfiguriert oder weitere Dateien eingebunden. Es lässt sich so eine Gruppierung der Änderungen vornehmen:

```xml
<databaseChangeLog
 xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd">

<include file="topic-file.xml" relativeToChangelogFile="true"/>
</databaseChangeLog>
```
Natürlich können hier auch mehrere Dateien eingebunden werden. Doch schauen wir uns das _topic-file.xml_ mal genauer an:

```xml
<databaseChangeLog
 xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd">

<include file="topic-1.0.0.xml" relativeToChangelogFile="true"/>
</databaseChangeLog>
```

Auch hier sind noch keine Änderungen direkt erfasst. Ich nutze diese Ebene, um die verschiedenen Versionen zu strukturieren. Damit lässt sich sehr einfach nachhalten, zu welchem Release welche Datenbankänderungen eingeflossen sind. Und zu guter letzt kommt die eigentliche Verwaltungsdatei _topic-1.0.0.xml_ (die ersten Nutzungen von Maven-Properties, die durch die Filterung von Maven dann ersetzt werden, habe ich noch hervorgehoben):

```xml
<databaseChangeLog
 xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd">

<changeSet id="topic-schema-create-h2" author="klenkes74">
 <preConditions>
 <dbms type="H2"/>
 <preConditions>

<comment>Creates the schema needed for the database.</comment>

<sql>CREATE SCHEMA IF NOT EXISTS ${db.schema};</sql>
 ...
 </changeSet>

<changeSet id="topic-initial" author="klenkes74">
 <comment>The initial data base tables.</comment>
 ...
 </changeSet>
</databaseChangeLog>
```
Über die eigentliche Syntax der Datei möchte ich mich nicht auslassen, sie ist auf der Webseite von Liquibase gut beschrieben und recht eingängig. Wichtig ist, dass ein Changeset nicht mehr verändert werden darf, sobald es einmal im zentralen SCM (git oder svn oder was immer ihr auch einsetzen mögt) eingecheckt wurde. Alle späteren Änderungen müssen als weitere Changesets veröffentlicht werden.

Diese Dateien lege ich meistens im Ordner _src/main/resources/ddl_ ab. Sollte ich Daten via Liquibase importieren (mit dieser Funktion kann man sich oft ein gesondertes Aufsetzen von z.B. dbunit sparen), dann liegen diese Dateien meist unter _src/main/resources/data_ für Daten, die auch in Produktion gebraucht werden (z.B. einen Admin-User in den Benutzertabellen) oder unter _src/test/resources/data_ für Testdaten. Apropos Testdaten. Hier hilft meine Changelog-Datei, die unter _src/test/resources/ddl_ liegt:

```xml
<databaseChangeLog
 xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd">

<!-- Import the real changelog -->
 <include file="${project.build.outputDirectory}/ddl/changelog-master.xml" relativeToChangelogFile="false" />
</databaseChangeLog>
```
Wie man schön sieht, binde ich hier die Änderungsdatei aus der Produktion ein. Außerdem könnte ich hier jetzt noch Testdaten-Importe durchführen.

## Die JPA-persistence.xml Konfigurationsdatei

Ich nutze unter src/test/resources/META-INF/persistence.xml eine besondere persistence.xml für Integrationstests. Man sieht hier gut die Nutzung der Maven-Properties für die eigentliche Konfiguration. Natürlich muss Maven so aufgesetzt sein, dass die Resourcen auch gefiltert werden.

```xml
<persistence xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns="http://xmlns.jcp.org/xml/ns/persistence"
 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
 http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd"
 version="2.1">

<persistence-unit name="tenant">
 <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

... meine Klassen sind hier nicht besonders interessant, daher weg damit ...

<properties>
 <property name="javax.persistence.jdbc.url" value="${jdbc.url}"/>
 <property name="javax.persistence.jdbc.user" value="${jdbc.user}"/>
 <property name="javax.persistence.jdbc.password" value="${jdbc.password}"/>
 <property name="javax.persistence.jdbc.driver" value="${jdbc.driver}"/>
 <property name="hibernate.show_sql" value="false"/>
 <property name="hibernate.format_sql" value="true"/>
 <property name="hibernate.dialect" value="${jdbc.dialect}"/>
 <property name="hibernate.hbm2ddl.auto" value="validate"/>
 <!-- Configuring Connection Pool -->
 <property name="hibernate.c3p0.min_size" value="5"/>
 <property name="hibernate.c3p0.max_size" value="20"/>
 <property name="hibernate.c3p0.timeout" value="500"/>
 <property name="hibernate.c3p0.max_statements" value="50"/>
 <property name="hibernate.c3p0.idle_test_period" value="2000"/>
 </properties>
 </persistence-unit>
</persistence>
```
## Einbinden des Liquibase-Plugins

Nachdem ich die Daten vorbereitet habe, kommt jetzt das Zusammenfügen per Maven. zuerst brauchen wir ja die Properties, die oben ersetzt werden und einige, die uns das Arbeiten mit dem Plugin erleichtern. Zu beachten ist die Zeile 6, in der die benötigte URL steht. Hier nutze ich die Maven-Property **${h2port}**. Diese wird bei mir vom Maven-Build-Helper gesetzt. Wer diesen nicht einsetzen will, sollte hier einen entsprechenden Port eintragen, sonst wird das nicht funktionieren:

```xml
...
<properties>
 ...
 <skipDatabaseSetup>${skipTests}</skipDatabaseSetup>
 <db.schema>TENANT</db.schema>
 <database.changelog>${project.build.testOutputDirectory}/ddl/changelog-test.xml</database.changelog>

<jdbc.url>jdbc:h2:tcp://localhost:${h2port}/testdb;AUTO_RECONNECT=TRUE;DB_CLOSE_DELAY=-1</jdbc.url>
 <jdbc.driver>org.h2.Driver</jdbc.driver>
 <jdbc.user>sa</jdbc.user>
 <jdbc.password>password</jdbc.password>
 <jdbc.driver>org.h2.Driver</jdbc.driver>
 <jdbc.dialect>org.hibernate.dialect.H2Dialect</jdbc.dialect>

<liquibase-maven-plugin.version>3.5.2</liquibase-maven-plugin.version>

<h2.version>1.3.176</h2.version>
 ...
 </properties>
...
```
Und nun die Plugin-Definition innerhalb der Builddefinition des pom-Files:

```xml
...
 <build>
 ...
 <plugins>
 ...
 <plugin>
 <groupId>org.liquibase</groupId>
 <artifactId>liquibase-maven-plugin</artifactId>
 <version>${liquibase-maven-plugin.version}</version>

<configuration>
 <changeLogFile>${database.changelog}</changeLogFile>
 <driver>${jdbc.driver}</driver>
 <url>${jdbc.url}</url>
 <username>${jdbc.user}</username>
 <password>${jdbc.password}</password>
 <verbose>true</verbose>
 <logging>warning</logging>
 <promptOnNonLocalDatabase>false</promptOnNonLocalDatabase>
 <skip>${skipDatabaseSetup}</skip>
 <dropFirst>false</dropFirst>
 </configuration>
 <dependencies>
 <dependency>
 <groupId>com.h2database</groupId>
 <artifactId>h2</artifactId>
 <version>${h2.version}</version>
 </dependency>
 </dependencies>
 <executions>
 <execution>
 <id>update-db</id>
 <phase>pre-integration-test</phase>
 <configuration>
 <changeLogFile>${database.changelog}</changeLogFile>
 <driver>${jdbc.driver}</driver>
 <url>${jdbc.url}</url>
 <username>${jdbc.user}</username>
 <password>${jdbc.password}</password>
 <verbose>true</verbose>
 <logging>warning</logging>
 <promptOnNonLocalDatabase>false</promptOnNonLocalDatabase>
 <skip>${skipDatabaseSetup}</skip>
 <dropFirst>false</dropFirst>
 </configuration>
 <goals>
 <goal>update</goal>
 </goals>
 </execution>
 </executions>
 </plugin>
 ...
 </plugins>
 ...
 </build>
...
```
Und das war es eigentlich auch schon. Hat man alles richtig gemacht, wird nun in der Maven-Build-Phase _pre-integration-test_ die Datenbank via Liquibase upgedated. Wer sich das ganze im Zusammenspiel anschauen möchte, ist gerne eingeladen, mal in mein Projekt https://github.com/klenkes74/kp-office zu schauen. Dort habe ich ein Parent für JPA-Module ([kp-office-parent-adapter-jpa](https://github.com/klenkes74/kp-office/blob/master/kp-office-parents-root/kp-office-parent-adapter-jpa/pom.xml)) mit den globalen Definitionen (wie dem Plugin) und dann Modulen, die dies nutzen (z.B. [kp-office-base-adapter-jpa](https://github.com/klenkes74/kp-office/tree/master/kp-office-modules/kp-office-base-root/kp-office-base-adapter-jpa)), in denen dann die Changelog-Dateien und Testdaten für die eigentlichen Datenbanken liegen.
