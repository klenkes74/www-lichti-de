---
layout: page
title: Und wieder einmal ein neuer Cluster und Webseiten
thumbnail: /assets/thumbnail/rz-768x432.jpg
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
date: '2025-09-27 21:12:00 +0200'
categories:
- k8s
- Hosting
- Deutsch
tags:
- k8s
- Hetzner
- Hosting
- jekyll
- wordpress
comments: []
---
Und wieder war es wieder Zeit, weiterzuziehen. Nachdem die OKD-Updates sehr komplex wurden, habe ich mich für einen neuen Cluster entschieden.

Diesmal ist es ein einfacher k8s-Cluster. 
Wieder bei Hetzner.
Aber diesmal nicht als VMs auf einer einzelnen Maschine, sondern echte Maschinen.
Die Workernodes sind drei Maschinen aus der Server-Börse mit jeweils 12 Cores und 128 GB RAM.
Die Master sind drei Cloud-Maschinen CPX21 mit jeweils 3 vCPUs und 4 GB RAM.

Meine Webseiten [www.lichti.de](https://www.lichti.de) und [www.paladins-inn.de](https://www.paladins-inn.de) sind (oder werden) damit auch umgestellt.
Bis jetzt sind es Wordpress-Seiten, die jedoch ständige Arbeit für Wordpress-Updates benötigen.
Daher stelle ich diese jetzt auf statische Blogs auf Jekyll-Basis um.
Für [www.lichti.de](https://www.lichti.de) habe ich das schon geschafft, [www.paladins-inn.de](https://www.paladins-inn.de) folgt in den nächsten Tagen.