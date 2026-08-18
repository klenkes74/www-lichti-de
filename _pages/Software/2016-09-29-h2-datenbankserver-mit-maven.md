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
Der Datenbankserver H2 ist ein beliebter Server während der Entwicklungsphase einer Software. Die Fähigkeit zu In-Memory-Datenbanken ist für viele Tests geradezu ideal. Wer jedoch die Datenbankstrukturen analysieren will, kommt um eine Datenbank mit Persistierung nicht herum. H2 kann natürlich das auch. Allerdings gestaltet sich das Starten/Stoppen für die Integrationstests etwas komplex. Natürlich gibt es entsprechende Plugins, aber auf einem CI-Server kommen so leicht Portkonflikte zustanden, wenn mehrere Builds gleichzeitig laufen. Ich beschreibe hier ein Setting, das den Maven-Build-Helper nutzt, um dies zu verhindern.

Das eigentliche Starten und Stoppen geht per Plugin ganz einfach. Allerdings mögen es CI-Systeme wie z.B. Jenkins nicht, wenn z.B. der Datenbankport hart vorgegeben ist. Da sind Konflikte einfach vorprogrammiert. Aber [Mojohaus](http://www.mojohaus.org/) (Nachfolger des Mojo-Projekts bei Codehaus) bietet auch hier Abhilfe mit dem [build-helper-maven-plugin](http://www.mojohaus.org/build-helper-maven-plugin/index.html).

```xml
 ...
 <build>
 <plugins>
 <plugin>
 <groupId>org.codehaus.mojo</groupId>
 <artifactId>build-helper-maven-plugin</artifactId>
 <version>1.12</version>
 <executions>
 <execution>
 <id>reserve-ports</id>
 <phase>generate-sources</phase>
 <goals>
 <goal>reserve-network-port</goal>
 </goals>
 <configuration>
 <portNames>
 <portName>h2port</portName>
 </portNames>
 </configuration>
 </execution>
 </executions>
 </plugin>
 ...
 </plugins>
 ...
 </build>
 ...
```
Mit diesem Port (der jetzt als Maven-Property **${h2port}** verfügbar ist) kann man nun das eigentliche Plugin konfigurieren. Wichtig ist, dass das Plugin die gleiche Version der H2-Datenbank nutzt, wie die Anwendung. Sonst kommen unschöne Fehler vor, die sich schwer debuggen lassen. Daher habe ich die Versionen wieder einmal als Properties ausgelagert:

```xml
 ...
 <properties>
 <h2.version>1.3.176</h2.version>
 <h2-maven-plugin.version>1.0</h2-maven-plugin.version>
 </properties>
 ...
 <build>
 ...
 <plugins>
 ...
 <plugin>
 <groupId>com.edugility</groupId>
 <artifactId>h2-maven-plugin</artifactId>
 <version>${h2-maven-plugin.version}</version>
 <configuration>
 <!--suppress MavenModelInspection --><!-- Will be set by the build-helper -->
 <port>${h2port}</port>
 <allowOthers>true</allowOthers>
 <baseDirectory>${project.basedir}/target/data</baseDirectory>
 <shutdownAllServers>true</shutdownAllServers>
 <trace>true</trace>
 </configuration>
 <executions>
 <execution>
 <id>Spawn a new H2 TCP server</id>
 <phase>pre-integration-test</phase>
 <goals>
 <goal>spawn</goal>
 </goals>
 </execution>
 <execution>
 <id>Stop a spawned H2 TCP server</id>
 <phase>post-integration-test</phase>
 <goals>
 <goal>stop</goal>
 </goals>
 </execution>
 </executions>
 <dependencies>
 <dependency>
 <groupId>com.h2database</groupId>
 <artifactId>h2</artifactId>
 <version>${h2.version}</version>
 </dependency>
 </dependencies>
 </plugin>
 ...
 </plugins>
 ...
 </build>
 ...
```
Damit wird nun der H2-Server vor den Integrationstests gestartet und danach gestoppt. Wichtig ist, dass die Integrationstests per failsafe ausgeführt werden, da der Stop durch die dynamische Zuweisung des Ports nicht mehr automatisiert durchgeführt werden kann, sobald der Mavenprozess abgebrochen ist. Ein Build-Abbruch während der Integrationstests führt also zu einem hängendem Datenbankserver auf einem automatisch generierten Port.

Im Zusammenspiel mit einem Datenbankversionierungssystem wie z.B. Liquibase lässt sich nun viel in Maven [automatisieren](https://www.lichti.de/2016/09/29/datenbankversionierung-mit-liquibase-und-maven/).
