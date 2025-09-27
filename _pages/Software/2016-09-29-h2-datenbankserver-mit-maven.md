---
layout: page
status: publish
published: true
title: H2-Datenbankserver für Integrationstests mit Maven starten und stoppen
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 258
wordpress_url: https://www.lichti.de/?p=258
date: '2016-09-29 06:02:20 +0200'
date_gmt: '2016-09-29 04:02:20 +0200'
categories:
- Softwareschnipsel
- Deutsch
tags: []
comments: []
---
<p>Der Datenbankserver H2 ist ein beliebter Server während der Entwicklungsphase einer Software. Die Fähigkeit zu In-Memory-Datenbanken ist für viele Tests geradezu ideal. Wer jedoch die Datenbankstrukturen analysieren will, kommt um eine Datenbank mit Persistierung nicht herum. H2 kann natürlich das auch. Allerdings gestaltet sich das Starten/Stoppen für die Integrationstests etwas komplex. Natürlich gibt es entsprechende Plugins, aber auf einem CI-Server kommen so leicht Portkonflikte zustanden, wenn mehrere Builds gleichzeitig laufen. Ich beschreibe hier ein Setting, das den Maven-Build-Helper nutzt, um dies zu verhindern.</p>
<p><!--more--></p>
<p>Das eigentliche Starten und Stoppen geht per Plugin ganz einfach. Allerdings mögen es CI-Systeme wie z.B. Jenkins nicht, wenn z.B. der Datenbankport hart vorgegeben ist. Da sind Konflikte einfach vorprogrammiert. Aber <a href="http://www.mojohaus.org/">Mojohaus</a> (Nachfolger des Mojo-Projekts bei Codehaus) bietet auch hier Abhilfe mit dem <a href="http://www.mojohaus.org/build-helper-maven-plugin/index.html">build-helper-maven-plugin</a>.</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
  ...<br />
  &lt;build&gt;<br />
    &lt;plugins&gt;<br />
      &lt;plugin&gt;<br />
        &lt;groupId&gt;org.codehaus.mojo&lt;/groupId&gt;<br />
        &lt;artifactId&gt;build-helper-maven-plugin&lt;/artifactId&gt;<br />
        &lt;version&gt;1.12&lt;/version&gt;<br />
        &lt;executions&gt;<br />
          &lt;execution&gt;<br />
            &lt;id&gt;reserve-ports&lt;/id&gt;<br />
            &lt;phase&gt;generate-sources&lt;/phase&gt;<br />
            &lt;goals&gt;<br />
              &lt;goal&gt;reserve-network-port&lt;/goal&gt;<br />
            &lt;/goals&gt;<br />
            &lt;configuration&gt;<br />
              &lt;portNames&gt;<br />
                 &lt;portName&gt;h2port&lt;/portName&gt;<br />
              &lt;/portNames&gt;<br />
            &lt;/configuration&gt;<br />
         &lt;/execution&gt;<br />
       &lt;/executions&gt;<br />
      &lt;/plugin&gt;<br />
      ...<br />
    &lt;/plugins&gt;<br />
    ...<br />
  &lt;/build&gt;<br />
  ...<br />
[/xml]</p>
<p>Mit diesem Port (der jetzt als Maven-Property <strong>${h2port}</strong> verfügbar ist) kann man nun das eigentliche Plugin konfigurieren. Wichtig ist, dass das Plugin die gleiche Version der H2-Datenbank nutzt, wie die Anwendung. Sonst kommen unschöne Fehler vor, die sich schwer debuggen lassen. Daher habe ich die Versionen wieder einmal als Properties ausgelagert:</p>
<p>[xml autolinks="false" collapse="false" firstline="1" gutter="true" highlight="3-4,14,17,43" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
  ...<br />
  &lt;properties&gt;<br />
    &lt;h2.version&gt;1.3.176&lt;/h2.version&gt;<br />
    &lt;h2-maven-plugin.version&gt;1.0&lt;/h2-maven-plugin.version&gt;<br />
  &lt;/properties&gt;<br />
  ...<br />
  &lt;build&gt;<br />
    ...<br />
    &lt;plugins&gt;<br />
      ...<br />
      &lt;plugin&gt;<br />
        &lt;groupId&gt;com.edugility&lt;/groupId&gt;<br />
        &lt;artifactId&gt;h2-maven-plugin&lt;/artifactId&gt;<br />
        &lt;version&gt;${h2-maven-plugin.version}&lt;/version&gt;<br />
        &lt;configuration&gt;<br />
          &lt;!--suppress MavenModelInspection --&gt;&lt;!-- Will be set by the build-helper --&gt;<br />
          &lt;port&gt;${h2port}&lt;/port&gt;<br />
          &lt;allowOthers&gt;true&lt;/allowOthers&gt;<br />
          &lt;baseDirectory&gt;${project.basedir}/target/data&lt;/baseDirectory&gt;<br />
          &lt;shutdownAllServers&gt;true&lt;/shutdownAllServers&gt;<br />
          &lt;trace&gt;true&lt;/trace&gt;<br />
        &lt;/configuration&gt;<br />
        &lt;executions&gt;<br />
          &lt;execution&gt;<br />
            &lt;id&gt;Spawn a new H2 TCP server&lt;/id&gt;<br />
            &lt;phase&gt;pre-integration-test&lt;/phase&gt;<br />
            &lt;goals&gt;<br />
              &lt;goal&gt;spawn&lt;/goal&gt;<br />
            &lt;/goals&gt;<br />
          &lt;/execution&gt;<br />
          &lt;execution&gt;<br />
            &lt;id&gt;Stop a spawned H2 TCP server&lt;/id&gt;<br />
            &lt;phase&gt;post-integration-test&lt;/phase&gt;<br />
            &lt;goals&gt;<br />
              &lt;goal&gt;stop&lt;/goal&gt;<br />
            &lt;/goals&gt;<br />
          &lt;/execution&gt;<br />
        &lt;/executions&gt;<br />
        &lt;dependencies&gt;<br />
          &lt;dependency&gt;<br />
            &lt;groupId&gt;com.h2database&lt;/groupId&gt;<br />
            &lt;artifactId&gt;h2&lt;/artifactId&gt;<br />
            &lt;version&gt;${h2.version}&lt;/version&gt;<br />
          &lt;/dependency&gt;<br />
        &lt;/dependencies&gt;<br />
      &lt;/plugin&gt;<br />
      ...<br />
    &lt;/plugins&gt;<br />
    ...<br />
   &lt;/build&gt;<br />
   ...<br />
[/xml]</p>
<p>Damit wird nun der H2-Server vor den Integrationstests gestartet und danach gestoppt. Wichtig ist, dass die Integrationstests per failsafe ausgeführt werden, da der Stop durch die dynamische Zuweisung des Ports nicht mehr automatisiert durchgeführt werden kann, sobald der Mavenprozess abgebrochen ist. Ein Build-Abbruch während der Integrationstests führt also zu einem hängendem Datenbankserver auf einem automatisch generierten Port.</p>
<p>Im Zusammenspiel mit einem Datenbankversionierungssystem wie z.B. Liquibase lässt sich nun viel in Maven <a href="https://www.lichti.de/2016/09/29/datenbankversionierung-mit-liquibase-und-maven/">automatisieren</a>.</p>
