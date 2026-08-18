---
layout: page
status: publish
published: true
title: Logging in Java
thumbnail: /assets/thumbnail/oks4yPK-380x500.jpg
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 312
wordpress_url: https://www.lichti.de/?p=312
date: '2017-01-03 13:46:39 +0100'
date_gmt: '2017-01-03 12:46:39 +0100'
categories:
- Softwareschnipsel
- Deutsch
tags:
- Java
- Software
comments: []
---
Der Artikel "[Is Standard Java Logging Dead?](https://dzone.com/articles/is-standard-java-logging-dead)" auf [dzone.com](https://dzone.com/) hat mich animiert, wieder einmal ein paar Gedanken zu Papier zu bringen. Logging ist immer ein nettes Thema in den Projekten. Meistens wird wenig dazu gesagt oder vereinbart. Jeder Programmierer zieht sein Ding durch. Doch wenn Logging wirklich helfen soll, dann muss man sich ein paar Gedanken machen.

Zu allererst muss man sich klarmachen, für wen das Logging ist und wie es ihn unterstützen soll. Sofort kommen zwei Adressaten in den Sinn. Natürlich der Entwickler. Er braucht Informationen während der Entwicklung und die Einführungsphase der Software. Und dann braucht natürlich der Admin in der Produktion Logmeldungen, die ihn in das System sehen lassen.

Aber es gibt noch weitere Adressaten. So kann zum Beispiel ein Abrechnungssystem auf entsprechende Daten angewiesen sein. Oder die Governance-Abteilung. Moderne Frameworks erlauben, Logging-Events nach diversen Regeln zu interpretieren.

Aber jetzt beginnt das Problem. Der Programmierer wird das Logging so bauen, dass er mit dem Wissen, was er programmiert hat, schnell Informationen bekommt. Dummerweise hat sein Nachfolger im Projekt oder der Administrator dieses Wissen nicht mehr. Und fragt sich, warum _hier_ geloggt wird und _da_ nicht. Mit diesem Entwicklerlogging kann der Admin und erst recht das Billing nichts anfangen. Daher müssen Logginganforderungen auch als nicht-funktionale Anforderungen erfasst werden. Wenn man ein System aus Micro-Services zusammensetzt wird Operations sehr genau definieren müssen, wie es Logging-Events erhalten will, damit diese zentral verarbeitet und aufbereitet werden können - sonst müssen die Admins hunderte Logfiles auswerten und die einzelnen Geschäftsvorfälle durch die Systeme verfolgen. Eine nicht zu leistende Aufgabe.
