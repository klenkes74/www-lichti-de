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
<p>SQL-Datenbanken gehören trotz der großen Aufmerksamkeit für NOSQL zum Brot-und-Butter-Handwerkzeug eines Softwareentwicklers. Und zumindest im Unternehmensumfeld wird des wohl noch lange so bleiben, denn dort setzen sich neue Konzepte nur langsam durch. Doch schon während der Softwareentwicklung stößt man im Team auf das Problem, dass Änderungen an den Datenstrukturen verwaltet werden müssen. Die Kollegen brauchen bei Softwareänderungen auch geänderte Testdatenbanken, man will auch nachhalten, welcher Commit welche Datenbankänderung nach sich zog.</p>
<p>Hier kommen dann Datenbankversionierungstools wie <a href="ttps://flywaydb.org">Flyway</a> oder eben <a href="http://www.liquibase.org/">Liquibase</a> ins Spiel. Flyway habe ich uns vor langer Zeit nur kurz angeschaut, aber eine persönliche Präferenz des für das damalige Projekt verantwortlichen Softwarearchitekten für XML-basierte Tools hat mich dann zu Liquibase geführt. Wie es in ein Mavenprojekt integriert werden kann, damit Integrationstests immer auf eine aktuelle Datenbank zugreifen können, will ich im folgenden beschreiben.</p>
<p><!--more--></p>
<p>Ich werde hier nicht näher darauf eingehen, wie man zum Beispiel einen <a href="https://www.lichti.de/2016/09/29/h2-datenbankserver-mit-maven/">H2-Server in einen Maven-Integrationstest</a> einbauen kann. Ich gehe von einem laufenden Server aus, um dann die Datenbankstruktur validieren und updaten zu können.</p>
<h2>Die Verwaltung der Datenbankänderungen</h2>
<p>Man braucht natürlich die Dateien, die die Datenbank und ihre Änderungen beschreiben. Ich habe mir Angewöhnt, nicht erst beim 1. Update in der Produktion mit Liquibase zu arbeiten sondern bereits in der Entwicklung damit zu starten. Im Team kann man so auch leicht Strukturänderungen an den Datenbanken übernehmen und damit kommunizieren. Außerdem werden sie so im Sourcecodemanagement-Tool verwaltet und können (und sollten!) Bestandteil des Code-Review im Team werden. Ich fange meist mit einer zentralen Changelog-Datei an, die dann weitere Dateien einbindet. Je nach Komplexität werden in den eingebundenen Dateien direkt Datenbankänderungen konfiguriert oder weitere Dateien eingebunden. Es lässt sich so eine Gruppierung der Änderungen vornehmen:</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" highlight="6" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
&lt;databaseChangeLog<br />
        xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;<br />
        xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;<br />
        xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd&quot;&gt;</p>
<p>    &lt;include file=&quot;topic-file.xml&quot; relativeToChangelogFile=&quot;true&quot;/&gt;<br />
&lt;/databaseChangeLog&gt;<br />
[/xml]</p>
<p>Natürlich können hier auch mehrere Dateien eingebunden werden. Doch schauen wir uns das <em>topic-file.xml</em> mal genauer an:</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" highlight="6" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
&lt;databaseChangeLog<br />
        xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;<br />
        xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;<br />
        xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd&quot;&gt;</p>
<p>    &lt;include file=&quot;topic-1.0.0.xml&quot; relativeToChangelogFile=&quot;true&quot;/&gt;<br />
&lt;/databaseChangeLog&gt;[/xml]</p>
<p>Auch hier sind noch keine Änderungen direkt erfasst. Ich nutze diese Ebene, um die verschiedenen Versionen zu strukturieren. Damit lässt sich sehr einfach nachhalten, zu welchem Release welche Datenbankänderungen eingeflossen sind. Und zu guter letzt kommt die eigentliche Verwaltungsdatei <em>topic-1.0.0.xml</em> (die ersten Nutzungen von Maven-Properties, die durch die Filterung von Maven dann ersetzt werden, habe ich noch hervorgehoben):</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" hightlight="13,16,23" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
&lt;databaseChangeLog<br />
        xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;<br />
        xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;<br />
        xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd&quot;&gt;</p>
<p>    &lt;changeSet id=&quot;topic-schema-create-h2&quot; author=&quot;klenkes74&quot;&gt;<br />
        &lt;preConditions&gt;<br />
            &lt;dbms type=&quot;H2&quot;/&gt;<br />
        &lt;preConditions&gt;</p>
<p>        &lt;comment&gt;Creates the schema needed for the database.&lt;/comment&gt;</p>
<p>        &lt;sql&gt;CREATE SCHEMA IF NOT EXISTS ${db.schema};&lt;/sql&gt;<br />
        ...<br />
    &lt;/changeSet&gt;</p>
<p>    &lt;changeSet id=&quot;topic-initial&quot; author=&quot;klenkes74&quot;&gt;<br />
        &lt;comment&gt;The initial data base tables.&lt;/comment&gt;<br />
        ...<br />
    &lt;/changeSet&gt;<br />
&lt;/databaseChangeLog&gt;<br />
[/xml]</p>
<p>Über die eigentliche Syntax der Datei möchte ich mich nicht auslassen, sie ist auf der Webseite von Liquibase gut beschrieben und recht eingängig. Wichtig ist, dass ein Changeset nicht mehr verändert werden darf, sobald es einmal im zentralen SCM (git oder svn oder was immer ihr auch einsetzen mögt) eingecheckt wurde. Alle späteren Änderungen müssen als weitere Changesets veröffentlicht werden.</p>
<p>Diese Dateien lege ich meistens im Ordner <em>src/main/resources/ddl</em> ab. Sollte ich Daten via Liquibase importieren (mit dieser Funktion kann man sich oft ein gesondertes Aufsetzen von z.B. dbunit sparen), dann liegen diese Dateien meist unter <em>src/main/resources/data</em> für Daten, die auch in Produktion gebraucht werden (z.B. einen Admin-User in den Benutzertabellen) oder unter <em>src/test/resources/data</em> für Testdaten. Apropos Testdaten. Hier hilft meine Changelog-Datei, die unter <em>src/test/resources/ddl</em> liegt:</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" highlight="7" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
&lt;databaseChangeLog<br />
    xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;<br />
    xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;<br />
    xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd&quot;&gt;</p>
<p>    &lt;!-- Import the real changelog --&gt;<br />
    &lt;include file=&quot;${project.build.outputDirectory}/ddl/changelog-master.xml&quot; relativeToChangelogFile=&quot;false&quot; /&gt;<br />
&lt;/databaseChangeLog&gt;<br />
[/xml]</p>
<p>Wie man schön sieht, binde ich hier die Änderungsdatei aus der Produktion ein. Außerdem könnte ich hier jetzt noch Testdaten-Importe durchführen.</p>
<h2>Die JPA-persistence.xml Konfigurationsdatei</h2>
<p>Ich nutze unter src/test/resources/META-INF/persistence.xml eine besondere persistence.xml für Integrationstests. Man sieht hier gut die Nutzung der Maven-Properties für die eigentliche Konfiguration. Natürlich muss Maven so aufgesetzt sein, dass die Resourcen auch gefiltert werden.</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" highlight="13-16,19" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
&lt;persistence xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;<br />
             xmlns=&quot;http://xmlns.jcp.org/xml/ns/persistence&quot;<br />
             xsi:schemaLocation=&quot;http://xmlns.jcp.org/xml/ns/persistence<br />
             http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd&quot;<br />
             version=&quot;2.1&quot;&gt;</p>
<p>    &lt;persistence-unit name=&quot;tenant&quot;&gt;<br />
        &lt;provider&gt;org.hibernate.jpa.HibernatePersistenceProvider&lt;/provider&gt;</p>
<p>        ... meine Klassen sind hier nicht besonders interessant, daher weg damit ...</p>
<p>        &lt;properties&gt;<br />
            &lt;property name=&quot;javax.persistence.jdbc.url&quot; value=&quot;${jdbc.url}&quot;/&gt;<br />
            &lt;property name=&quot;javax.persistence.jdbc.user&quot; value=&quot;${jdbc.user}&quot;/&gt;<br />
            &lt;property name=&quot;javax.persistence.jdbc.password&quot; value=&quot;${jdbc.password}&quot;/&gt;<br />
            &lt;property name=&quot;javax.persistence.jdbc.driver&quot; value=&quot;${jdbc.driver}&quot;/&gt;<br />
            &lt;property name=&quot;hibernate.show_sql&quot; value=&quot;false&quot;/&gt;<br />
            &lt;property name=&quot;hibernate.format_sql&quot; value=&quot;true&quot;/&gt;<br />
            &lt;property name=&quot;hibernate.dialect&quot; value=&quot;${jdbc.dialect}&quot;/&gt;<br />
            &lt;property name=&quot;hibernate.hbm2ddl.auto&quot; value=&quot;validate&quot;/&gt;<br />
            &lt;!-- Configuring Connection Pool --&gt;<br />
            &lt;property name=&quot;hibernate.c3p0.min_size&quot; value=&quot;5&quot;/&gt;<br />
            &lt;property name=&quot;hibernate.c3p0.max_size&quot; value=&quot;20&quot;/&gt;<br />
            &lt;property name=&quot;hibernate.c3p0.timeout&quot; value=&quot;500&quot;/&gt;<br />
            &lt;property name=&quot;hibernate.c3p0.max_statements&quot; value=&quot;50&quot;/&gt;<br />
            &lt;property name=&quot;hibernate.c3p0.idle_test_period&quot; value=&quot;2000&quot;/&gt;<br />
        &lt;/properties&gt;<br />
    &lt;/persistence-unit&gt;<br />
&lt;/persistence&gt;<br />
[/xml]</p>
<h2>Einbinden des Liquibase-Plugins</h2>
<p>Nachdem ich die Daten vorbereitet habe, kommt jetzt das Zusammenfügen per Maven. zuerst brauchen wir ja die Properties, die oben ersetzt werden und einige, die uns das Arbeiten mit dem Plugin erleichtern. Zu beachten ist die Zeile 6, in der die benötigte URL steht. Hier nutze ich die Maven-Property <strong>${h2port}</strong>. Diese wird bei mir vom Maven-Build-Helper gesetzt. Wer diesen nicht einsetzen will, sollte hier einen entsprechenden Port eintragen, sonst wird das nicht funktionieren:</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" highlight="8" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
...<br />
&lt;properties&gt;<br />
    ...<br />
    &lt;skipDatabaseSetup&gt;${skipTests}&lt;/skipDatabaseSetup&gt;<br />
    &lt;db.schema&gt;TENANT&lt;/db.schema&gt;<br />
    &lt;database.changelog&gt;${project.build.testOutputDirectory}/ddl/changelog-test.xml&lt;/database.changelog&gt;</p>
<p>    &lt;jdbc.url&gt;jdbc:h2:tcp://localhost:${h2port}/testdb;AUTO_RECONNECT=TRUE;DB_CLOSE_DELAY=-1&lt;/jdbc.url&gt;<br />
    &lt;jdbc.driver&gt;org.h2.Driver&lt;/jdbc.driver&gt;<br />
    &lt;jdbc.user&gt;sa&lt;/jdbc.user&gt;<br />
    &lt;jdbc.password&gt;password&lt;/jdbc.password&gt;<br />
    &lt;jdbc.driver&gt;org.h2.Driver&lt;/jdbc.driver&gt;<br />
    &lt;jdbc.dialect&gt;org.hibernate.dialect.H2Dialect&lt;/jdbc.dialect&gt;</p>
<p>    &lt;liquibase-maven-plugin.version&gt;3.5.2&lt;/liquibase-maven-plugin.version&gt;</p>
<p>    &lt;h2.version&gt;1.3.176&lt;/h2.version&gt;<br />
    ...<br />
  &lt;/properties&gt;<br />
...<br />
[/xml]</p>
<p>Und nun die Plugin-Definition innerhalb der Builddefinition des pom-Files:</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" highlight="12-16,20,27,35-39,43" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
...<br />
  &lt;build&gt;<br />
    ...<br />
    &lt;plugins&gt;<br />
      ...<br />
      &lt;plugin&gt;<br />
        &lt;groupId&gt;org.liquibase&lt;/groupId&gt;<br />
        &lt;artifactId&gt;liquibase-maven-plugin&lt;/artifactId&gt;<br />
        &lt;version&gt;${liquibase-maven-plugin.version}&lt;/version&gt;</p>
<p>        &lt;configuration&gt;<br />
          &lt;changeLogFile&gt;${database.changelog}&lt;/changeLogFile&gt;<br />
          &lt;driver&gt;${jdbc.driver}&lt;/driver&gt;<br />
          &lt;url&gt;${jdbc.url}&lt;/url&gt;<br />
          &lt;username&gt;${jdbc.user}&lt;/username&gt;<br />
          &lt;password&gt;${jdbc.password}&lt;/password&gt;<br />
          &lt;verbose&gt;true&lt;/verbose&gt;<br />
          &lt;logging&gt;warning&lt;/logging&gt;<br />
          &lt;promptOnNonLocalDatabase&gt;false&lt;/promptOnNonLocalDatabase&gt;<br />
          &lt;skip&gt;${skipDatabaseSetup}&lt;/skip&gt;<br />
          &lt;dropFirst&gt;false&lt;/dropFirst&gt;<br />
        &lt;/configuration&gt;<br />
        &lt;dependencies&gt;<br />
          &lt;dependency&gt;<br />
            &lt;groupId&gt;com.h2database&lt;/groupId&gt;<br />
            &lt;artifactId&gt;h2&lt;/artifactId&gt;<br />
            &lt;version&gt;${h2.version}&lt;/version&gt;<br />
          &lt;/dependency&gt;<br />
        &lt;/dependencies&gt;<br />
        &lt;executions&gt;<br />
          &lt;execution&gt;<br />
            &lt;id&gt;update-db&lt;/id&gt;<br />
            &lt;phase&gt;pre-integration-test&lt;/phase&gt;<br />
            &lt;configuration&gt;<br />
              &lt;changeLogFile&gt;${database.changelog}&lt;/changeLogFile&gt;<br />
              &lt;driver&gt;${jdbc.driver}&lt;/driver&gt;<br />
              &lt;url&gt;${jdbc.url}&lt;/url&gt;<br />
              &lt;username&gt;${jdbc.user}&lt;/username&gt;<br />
              &lt;password&gt;${jdbc.password}&lt;/password&gt;<br />
              &lt;verbose&gt;true&lt;/verbose&gt;<br />
              &lt;logging&gt;warning&lt;/logging&gt;<br />
              &lt;promptOnNonLocalDatabase&gt;false&lt;/promptOnNonLocalDatabase&gt;<br />
              &lt;skip&gt;${skipDatabaseSetup}&lt;/skip&gt;<br />
              &lt;dropFirst&gt;false&lt;/dropFirst&gt;<br />
            &lt;/configuration&gt;<br />
            &lt;goals&gt;<br />
              &lt;goal&gt;update&lt;/goal&gt;<br />
            &lt;/goals&gt;<br />
          &lt;/execution&gt;<br />
        &lt;/executions&gt;<br />
      &lt;/plugin&gt;<br />
     ...<br />
   &lt;/plugins&gt;<br />
 ...<br />
 &lt;/build&gt;<br />
...<br />
[/xml]</p>
<p>Und das war es eigentlich auch schon. Hat man alles richtig gemacht, wird nun in der Maven-Build-Phase <em>pre-integration-test</em> die Datenbank via Liquibase upgedated. Wer sich das ganze im Zusammenspiel anschauen möchte, ist gerne eingeladen, mal in mein Projekt https://github.com/klenkes74/kp-office zu schauen. Dort habe ich ein Parent für JPA-Module (<a href="https://github.com/klenkes74/kp-office/blob/master/kp-office-parents-root/kp-office-parent-adapter-jpa/pom.xml">kp-office-parent-adapter-jpa</a>) mit den globalen Definitionen (wie dem Plugin) und dann Modulen, die dies nutzen (z.B. <a href="https://github.com/klenkes74/kp-office/tree/master/kp-office-modules/kp-office-base-root/kp-office-base-adapter-jpa">kp-office-base-adapter-jpa</a>), in denen dann die Changelog-Dateien und Testdaten für die eigentlichen Datenbanken liegen.</p>
