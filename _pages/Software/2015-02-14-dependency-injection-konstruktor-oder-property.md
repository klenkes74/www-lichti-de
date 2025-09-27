---
layout: page
status: publish
published: true
title: Dependency Injection - Konstruktor oder Property?
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 200
wordpress_url: https://www.lichti.de/?p=200
date: '2015-02-14 12:03:13 +0100'
date_gmt: '2015-02-14 11:03:13 +0100'
categories:
- Softwareschnipsel
- Deutsch
tags: []
comments: []
---
<p>Dependency Injection hat sich durchgesetzt. Das muss also nicht mehr diskutiert werden. Es gibt zwar noch alte Projekte, die anders arbeiten (davon gibt es massig, denn auch in der Vergangenheit waren wir Programmierer ja nicht faul sondern haben Software geschrieben) - aber neuere Entwicklungen kommen selten ohne aus.</p>
<p>Leider gibt es noch oft Diskussionen, ob die DI via Property (also Setter-Methoden) oder den Konstruktor gehen soll. In den meisten Softwareprojekten sieht man ausschließlich Property-Injection (also einen Default-Konstruktor sowie ganz viele Setter). Dabei wird leider eine aus meiner Sicht grundlegende Regel vergessen: nach dem Konstruktor soll das Objekt minimal lauffähig sein. Dass heißt, dass alle <em>verpflichtenden</em> Informationen vorhanden sein müssen.</p>
<p>Oder anders ausgedrückt: in den seltensten Fällen ist ein Default-Konstrutor ausreichend für ein funktionierendes Objekt. Demnach spricht vieles für eine konstruktorbasierte DI.</p>
<p>Aber auch hier greift es zu kurz: der Konstruktor sollte ein <em>minimal lauffähiges Objekt</em> erzeugen. Und einige DI-Dependencies werden hier nicht benötigt. Diese Abhängigkeiten sollen also nicht in den Konstruktor. Und da spricht ja auch nichts dagegen: die <em>absolut notwendigen Abhängigkeiten</em> kommen in den <em>Konstruktor</em>, die <em>optionalen Abhängigkeiten</em> in <em>Properties</em>.</p>
<p>Oft wird angeführt, dass das weit verbreitete (und oft auch von mir genutzte) Spring-Framework keine Konstruktor-DI unterstützen würde oder die Entwickler von Konstruktor-DI abraten würden, aber wie der Blogeintrag <a title="Setter injection versus constructor injection and the use of @Required" href="http://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required/">http://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required/</a> schon 2007 beschrieb, sehen das unsere  Kollegen von Pivotal auch so wie hier beschrieben.</p>
<p>Voilà, das Problem ist sauber gelöst. Ohne Religionskrieg.</p>
