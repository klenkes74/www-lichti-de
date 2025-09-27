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
<p><!-- wp:paragraph --></p>
<p>Vor über 4 Jahren habe ich bereits im Beitrag <a href="https://www.lichti.de/2017/01/03/logging-in-java/">Logging in Java</a> ein paar grundlegende Gedanken zum Thema Logging geäußert. Dabei bin ich recht unspezifisch geblieben und habe nur ein paar grundlegende Ideen geäußert. Diesmal will ich etwas konkreter werden.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:more --><br />
<!--more--><br />
<!-- /wp:more --></p>
<p><!-- wp:paragraph --></p>
<p>Erstmal wie man Logifles schreibt. Hier nutze ich inzwischen immer <a rel="noreferrer noopener" href="http://slf4j.org/" target="_blank">Slf4j</a>. Mit dieser Fassade kann das eigentliche Logging über alle Logframeworks geleitet werden, ohne dass Änderungen am Quellcode notwendig werden. Es ist ein No-Brainer. Alles Logging über Slf4j, keine Diskussion darüber. Jede Klasse bekommt entweder einen statischen Logger (z.B über die Templating-Funktion der genutzten IDE bei der Erstellung des Files) oder - deutlich eleganter - mittels Der Annotation <kbd>@Slf4j</kbd> via <a rel="noreferrer noopener" href="https://projectlombok.org/" target="_blank"><kbd>lombok</kbd></a>. Eigentlich ist es egal, aber ich sage wieder: nutzt lombok. Punkt.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Nachdem das geklärt ist, wird es jetzt etwas diffiziler. Hier kommt jetzt das Fingerspitzengefühl. Ich orientiere mich an den üblichen Logleveln, die via Slf4j addressiert werden können und dann auf die entsprechenden Loglevel der Frameworks verteilt werden:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:table {"className":"is-style-stripes"} --></p>
<figure class="wp-block-table is-style-stripes">
<table>
<thead>
<tr>
<th>Fehlerlevel</th>
<th class="has-text-align-left" data-align="left">Beschreibung</th>
<th class="has-text-align-left" data-align="left">Auswirkung auf 24/7 Betrieb</th>
</tr>
</thead>
<tbody>
<tr>
<td>Error</td>
<td class="has-text-align-left" data-align="left"><strong>Ein technischer Fehler ist passiert.</strong> Das System könnte dauerhaft gestört sein und ein Administrator sollte darauf reagieren.</td>
<td class="has-text-align-left" data-align="left">Hier ist ein Bereitschaftsfall gegeben.</td>
</tr>
<tr>
<td>Warn</td>
<td class="has-text-align-left" data-align="left"><strong>Ein technischer Fehler ist passiert.</strong> Ein einzelner Aufruf oder Batch wurde gestört und ein Administrator sollte sich das einmal anschauen.</td>
<td class="has-text-align-left" data-align="left">Hier ist keine sofortige Reaktion notwendig. Es reicht, dass sich ein Admin das zu normalen Arbeitszeiten anschaut.</td>
</tr>
<tr>
<td>Info</td>
<td class="has-text-align-left" data-align="left">Ein einzelner Aufrag oder Batch wurde bearbeitet. <strong>Auch fachliche Fehler werden auf diesem Level protokolliert.</strong> Wenn die Aufruffrequenz es zulässt, kann z.B. der Start und das Ende jedes Systemaufrufs protokolliert werden, sollte das Logvolumen dadurch zu hoch sein, wird nur das Ende quitiert.</td>
<td class="has-text-align-left" data-align="left">Kein Fehler. Das Logvolumen sollte möglichst gering sein und nur die eigentlichen Transaktionen sichtbar sein.</td>
</tr>
<tr>
<td>Debug</td>
<td class="has-text-align-left" data-align="left">Der Ablauf eines Auftrags oder Batches wird sichtbar. Mindestens der Aufruf eines Services wird zusätzlich zum Ende protokolliert (wenn es nicht sowieso schon auf Info-Level passiert).</td>
<td class="has-text-align-left" data-align="left">Wenn ein System auf Fehlverhalten geprüft wird, sollten die Debug-Meldungen dem 2nd-Level-Support helfen, den Systemstatus zu erfassen und möglicht den Fehler zu analysieren und zu beheben. Das Logvolumen ist deutlich erhöht.</td>
</tr>
<tr>
<td>Trace</td>
<td class="has-text-align-left" data-align="left">Der Ablauf eines Auftrags oder Batches wird detailiert protokolliert.</td>
<td class="has-text-align-left" data-align="left">Hier kann der komplette Ablauf detailiert verfolgt werden. Es handelt sich um massives Logging, das dem 3rd-Level oder Lieferanten des Systems eine vollständige Analyse ermöglicht.</td>
</tr>
</tbody>
<tfoot>
<tr>
<td></td>
<td class="has-text-align-left" data-align="left"></td>
<td class="has-text-align-left" data-align="left"></td>
</tr>
</tfoot>
</table>
</figure>
<p><!-- /wp:table --></p>
<p><!-- wp:paragraph --></p>
<p>Natürlich sind diese "Regeln" abhängig von der Erfahrung, man muss lernen, zu antizipieren, was dem Betrieb hilft. Im Gegensatz zu meinem Artikel von 2017 bin ich inzwischen nicht mehr der Meinung, dass der Softwareentwickler selbst ein Addressat des Loggings ist. Als Entwickler nutze ich den Debugger bei der Entwicklung, das Logfile gehört dem Betrieb.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>Sollte ein fachliches Logging gebraucht werden, kann man einen zusätzlichen Logger zusätlich definieren (z.B. als "bussiness" oder "bus" - dort können dann die fachlichen Logfiles geschrieben werden. Hier fällt normalerweise deutlich weniger Logfile an und es werden meist nur die Level <kbd>Error</kbd>, <kbd>Info</kbd> und <kbd>Debug</kbd> benötigt.</p>
<p><!-- /wp:paragraph --></p>
