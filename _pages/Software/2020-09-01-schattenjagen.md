---
layout: page
status: publish
published: true
title: Schattenjagen
thumbnail: /assets/thumbnail/mu4Ihiw-768x576.jpg
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 573
wordpress_url: https://www.lichti.de/?p=573
date: '2020-09-01 07:16:45 +0200'
date_gmt: '2020-09-01 05:16:45 +0200'
categories:
- Das Wort zum Sonntag
- Deutsch
tags: []
comments: []
---
<p><!-- wp:paragraph --></p>
<p>Heute will ich etwas über eine beliebte Beschäftigung von Softwareentwicklern schreiben: dem Schattenjagen.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Schon früh lernt man als Softwareentwickler, Fehler zuerst bei sich zu suchen, bevor man Probleme bei anderen sucht. Oft genug ist es Lernen-durch-Schmerzen. Man behauptet, eine Library oder Code eines Kollegen hat einen Bug und bekommt dann nachgewiesen, dass es der eigene Code war. Man lernt also, erstmal auszuschließen, dass es sich bei dem Problem um den eigenen Code handelt, bevor man ihn woanders sucht.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Aber manchmal (oder öfter, je nach der einen Fähigkeit) ist es halt doch ein Problem, das man nicht selbst verursacht hat. Aber bis man dahin kommt, habt man oft schon lange Zeit bei der Fehlersuche verbracht.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Mein aktuelles Beispiel war z.B. ein Fehler in Liquibase, der bei der Nutzung von mariadb eine seltsame Exception geworfen hat (https://liquibase.jira.com/browse/CORE-3457). Und ausgerechnet bei einem neuen Pet-Project hatte ich mich entschieden, eine MariaDB zu nutzen. Und lief auf den Fehler. Ich habe an meinen Changelogs für Liquibase herumgeschraubt und sie solange umgeschrieben, bis ich einfach mal ein leeres Changelog nutzte und den gleichen Fehler bekam.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Und diesmal brachte mich Google dann auf die Jira-Issue CORE-3457. Ich habe also nun frohen Mutes einen Github-Issue für Quarkus 1.7.1.Final geöffnet - der derzeit aktuellen Version. Und dann wollte ich diesen Issue auch gleich lösen und einen Pullrequest dafür öffnen - man will ja auch etwas für die OpenSource tun und nicht immer nur geiern.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Also quarkus geforkt, in die IDE geladen und dann erst gesehen, dass im Master bereits liquibase 4.0.0 genutzt wird - und der Fehler wurde bereits vor ein paar Tagen gefixt.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Und so lebte mein Bug-Issue bei Quarkus ganze 10 Minuten, bevor ich ihn wieder kleinlaut geschlossen habe. Im Moment baue ich gerade das Snapshot-Quarkus, da ich keine 20 Tage warten will, bis ich weitermachen kann mit meinem Projekt. Aber alles in allem, habe ich sicherlich 5 Stunden wieder mal einen Schatten gejagt ...</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p><small>Titelbild: Meme von <a href="https://dev.to/glsolaria">G.L Solaria&nbsp;</a> auf dev.to</small></p>
<p><!-- /wp:paragraph --></p>
