---
layout: page
status: publish
published: true
title: MTBF und MTTR - Hä?
thumbnail: /assets/thumbnail/rgbstock_-_decar66_-_mfr3l5w_-_zen_bridge-cropped-300x150.jpg
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
excerpt: Nachdem ich diese Diskussion in der letzten Zeit mehrfach geführt habe, gehe
  ich im Rahmen eines Block-Posts auf diese strukturelle Änderung vom Rechenzentrums-IT-Betrieb
  auf Cloud-IT-Betrieb ein. Ich mache es an Rechenzentrum und Cloud fest, obwohl es
  eigentlich ein Paradigmenwechsel ist, der durch die erhöhte Automatisierung im Betrieb
  von Anwendungen und Systemen bedingt ist.
wordpress_id: 868
wordpress_url: https://www.lichti.de/?p=868
date: '2021-07-18 18:11:34 +0200'
date_gmt: '2021-07-18 16:11:34 +0200'
categories:
- Allgemein
- Softwareschnipsel
- Das Wort zum Sonntag
- Deutsch
- Gedankensplitter
tags:
- mtbf
- mttr
- KPI
- IT-Betrieb
- Cloud
- Kennzahlen
comments: []
---
<p><!-- wp:paragraph --></p>
<p>Nachdem ich diese Diskussion in der letzten Zeit mehrfach geführt habe, gehe ich im Rahmen eines Block-Posts auf diese strukturelle Änderung vom Rechenzentrums-IT-Betrieb auf Cloud-IT-Betrieb ein. Ich mache es an Rechenzentrum und Cloud fest, obwohl es eigentlich ein Paradigmenwechsel ist, der durch die erhöhte Automatisierung im Betrieb von Anwendungen und Systemen bedingt ist.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:more --><br />
<!--more--><br />
<!-- /wp:more --></p>
<p><!-- wp:heading --></p>
<h2>Aber zuerst einmal: was ist MTBF und MTTR?</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Unter MTBF versteht man die Mean Time Between Failure - also die durchschnittliche Zeit zwischen zwei Ausfällen eines Systems.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Die MTTR oder Mean Time To Repair ist die durchschnittliche Zeit, die benötigt wird, um ein System bei einem Ausfall wieder zu reparieren (also verfügbar zu machen).</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Nur damit wir komplett sind gibt es auch nocht die MTTF, die Mean Time to Failure. Also die Durchnittszeit bis zum Versagen. Als Formel ließe ich das Verhältnis der drei Betriffe als <code>MTBF = MTTF + MTTR</code> beschreiben. Aber ich werde die MTTF hier nicht weiter betrachten.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>IT-Bertrieb in der Vergangenheit</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Früher wurde der IT-Betrieb rein kostenbasiert betrachtet. Er sollte eine definierte Leistung bei minimalen Kosten liefern. Und die definierte Leistung war der störungsfreie Betrieb der Systeme. Damit war die Optimierung der IT klar: die Vergrößerung der MTBF, Denn jede einzelne Störung hat Kosten verursacht. Die Idee ist ja auch nicht schlecht, aber jeder erinnert sich noch an das Rennen zu den 5x9 (also 99,999% Verfügbarkeit). Jede 9 nach dem Komma vervielfacht die Kosten.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Trotzdem war es das Ziel, die <strong>MTBF</strong> zu optimieren. Hierzu dienten die allzeit bekannten Lessons learned und Root Cause Analyse <strong>bei jedem einzelnen Vorfall</strong>.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>Was hat sich geändert?</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Die IT ist schneller geworden. Sie ist inzwischen ein zentraler Bestandteil vieler Unternehmen, seien es Serviceunternehmen, die ihre Dienste in Software abbilden (wie z.B. Versicherungen oder Banken bei Vertragsabschlüssen oder Schadensabwicklungen, die oft genug ohne menschliche Intervention seitens des Unternehmens erfolgen) oder um die Firmware von Geräten oder Backendservices der Handy-Apps.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Software ändert sich nicht mehr im Quartals- oder Monatsrhythmus. Oft wird wöchentlich, täglich oder gar mehrfach am Tag eine neue Softwareversion eingespielt (ok, Firmware ist hier noch nicht ganz so weit, aber IoT ebnet dem schnellen Wechsel auch hier den Weg).</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>Und wo kommt hier die MTTR ins Spiel?</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Um dem schnellen Wandel zu managen, gehen viele IT-Betreiber "in die Cloud" oder bauen eine eigene "private Cloud" auf. Aber in der Cloud ändert sich etwas: wurden vorher noch die Rechenzentren (angefangen von den Netzwerkkomponenten, den Servern, Betriebssystem bis hin zur Laufzeitumgebung) an die Anwendungen angepasst, so diktieren nun die Clouds die komplette Laufzeitumgebung und die Anwendungen werden entsprechend umgebaut oder neu entwickelt. Containerisierung ist hier das Zauberwort und Mittel der Stunde. Aber darüber soll es hier nicht gehen.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>Kontrollverlust</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Es ist etwas anderes passiert, dass viele übersehen haben: Einflussverlust. Die Anwendungen haben die Kontrolle über die Infrastruktur verloren. Bei einer "Public Cloud", also das vollständig virtuelle Rechenzentrum bei einem Cloud-Anbieter wie AWS, Azure oder Google, hat der Kunde nur noch extrem wenig Einfluss, was im Netzwerk, der RZ-Technik oder den Servern passiert. Während im eigenen Rechenzentrum meist schon vor dem Ausfall einer wichtigen Infrastrukturkomponente die Warnsignale angehen, bekommt man davon "in der Cloud" nichts mit. Aber "die Cloud" heißt nur "Computer eines anderen Unternehmens", sie ist keine abstrakte Konstruktion im Nirvana. Auch dort gibt es Ausfälle. Für den Nutzer kommen sie nur unvorhergesehen. Natürlich sind die Cloudbetreiber sehr gut im Betrieb der Komponenten, auch sie mögen keine Ausfälle. Aber zählt mal die beteiligten Komponenten bei einer cloudbasierten Anwendung und bei einer altertümlichen RZ-basierten Anwendungen. Man wird sehen, dass die Cloud als verteiltes System in fast allen Fällen komplexer ist als die alten "on premise"-Systeme.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>Der Winter.....Fehler kommt!</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Es wird also zu vorher nicht absehbaren Fehlern kommen. Fehler, deren Auftreten man nicht beeinflussen kann. Man hat die Kontrolle über die MTBF verloren. Was bleibt? Man muss die MTTR optimieren. Wenn man weiß, dass Fehler passieren und man keine Chance hat, sie wegzuoptimieren oder zumindest zu reduzieren, dann muss man lernen, diese Fehler möglichst schnell in den Griff zu bekommen.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Lessons Learned und die Root Cause Analyse sind übrigens nicht mitgestorben. Aber sie werden nicht jedes Mal gemacht. Erst wenn die gleiche Störung mehrfach auftaucht, oder der Eindruck entsteht, dass verschiedene Störungen einen gemeinsamen Grund haben, kommen sie zum Einsatz.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p><em>Die Königin MTBF ist tot, hoch lebe die neue Königin MTTR.</em></p>
<p><!-- /wp:paragraph --></p>
