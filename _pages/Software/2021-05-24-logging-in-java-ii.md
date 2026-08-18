---
layout: page
status: publish
published: true
title: Logging in Java II
thumbnail: /assets/thumbnail/oks4yPK-380x500.jpg
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 832
wordpress_url: https://www.lichti.de/?p=832
date: '2021-05-24 23:39:56 +0200'
date_gmt: '2021-05-24 21:39:56 +0200'
categories:
- Softwareschnipsel
- Deutsch
- Gedankensplitter
tags:
- Java
- Observability
- Logging
- slf4j
comments: []
---
Vor über 4 Jahren habe ich bereits im Beitrag [Logging in Java](./logging-in-java.html) ein paar grundlegende Gedanken zum Thema Logging geäußert. Dabei bin ich recht unspezifisch geblieben und habe nur ein paar grundlegende Ideen geäußert. Diesmal will ich etwas konkreter werden.

<!--more-->

Erstmal wie man Logifles schreibt. Hier nutze ich inzwischen immer [Slf4j](http://slf4j.org/). Mit dieser Fassade kann das eigentliche Logging über alle Logframeworks geleitet werden, ohne dass Änderungen am Quellcode notwendig werden. Es ist ein No-Brainer. Alles Logging über Slf4j, keine Diskussion darüber. Jede Klasse bekommt entweder einen statischen Logger (z.B über die Templating-Funktion der genutzten IDE bei der Erstellung des Files) oder - deutlich eleganter - mittels der Annotation `@Slf4j` via [`lombok`](https://projectlombok.org/). Eigentlich ist es egal, aber ich sage wieder: nutzt lombok. Punkt.

Nachdem das geklärt ist, wird es jetzt etwas diffiziler. Hier kommt jetzt das Fingerspitzengefühl. Ich orientiere mich an den üblichen Logleveln, die via Slf4j addressiert werden können und dann auf die entsprechenden Loglevel der Frameworks verteilt werden:

| Fehlerlevel | Beschreibung | Auswirkung auf 24/7 Betrieb |
| --- | --- | --- |
| Error | **Ein technischer Fehler ist passiert.** Das System könnte dauerhaft gestört sein und ein Administrator sollte darauf reagieren. | Hier ist ein Bereitschaftsfall gegeben. |
| Warn | **Ein technischer Fehler ist passiert.** Ein einzelner Aufruf oder Batch wurde gestört und ein Administrator sollte sich das einmal anschauen. | Hier ist keine sofortige Reaktion notwendig. Es reicht, dass sich ein Admin das zu normalen Arbeitszeiten anschaut. |
| Info | Ein einzelner Aufrag oder Batch wurde bearbeitet. **Auch fachliche Fehler werden auf diesem Level protokolliert.** Wenn die Aufruffrequenz es zulässt, kann z.B. der Start und das Ende jedes Systemaufrufs protokolliert werden, sollte das Logvolumen dadurch zu hoch sein, wird nur das Ende quitiert. | Kein Fehler. Das Logvolumen sollte möglichst gering sein und nur die eigentlichen Transaktionen sichtbar sein. |
| Debug | Der Ablauf eines Auftrags oder Batches wird sichtbar. Mindestens der Aufruf eines Services wird zusätzlich zum Ende protokolliert (wenn es nicht sowieso schon auf Info-Level passiert). | Wenn ein System auf Fehlverhalten geprüft wird, sollten die Debug-Meldungen dem 2nd-Level-Support helfen, den Systemstatus zu erfassen und möglicht den Fehler zu analysieren und zu beheben. Das Logvolumen ist deutlich erhöht. |
| Trace | Der Ablauf eines Auftrags oder Batches wird detailiert protokolliert. | Hier kann der komplette Ablauf detailiert verfolgt werden. Es handelt sich um massives Logging, das dem 3rd-Level oder Lieferanten des Systems eine vollständige Analyse ermöglicht. |

Natürlich sind diese "Regeln" abhängig von der Erfahrung, man muss lernen, zu antizipieren, was dem Betrieb hilft. Im Gegensatz zu meinem Artikel von 2017 bin ich inzwischen nicht mehr der Meinung, dass der Softwareentwickler selbst ein Addressat des Loggings ist. Als Entwickler nutze ich den Debugger bei der Entwicklung, das Logfile gehört dem Betrieb.

Sollte ein fachliches Logging gebraucht werden, kann man einen zusätzlichen Logger zusätlich definieren (z.B. als "bussiness" oder "bus" - dort können dann die fachlichen Logfiles geschrieben werden. Hier fällt normalerweise deutlich weniger Logfile an und es werden meist nur die Level `Error`, `Info` und `Debug` benötigt.
