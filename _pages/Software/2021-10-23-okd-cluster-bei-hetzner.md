---
layout: page
status: publish
published: true
title: OKD Cluster bei Hetzner
thumbnail: /assets/thumbnail/rz-768x432.jpg
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 875
wordpress_url: https://www.lichti.de/?p=875
date: '2021-10-23 21:18:11 +0200'
date_gmt: '2021-10-23 19:18:11 +0200'
categories:
- OpenShift
- Deutsch
tags:
- OKD
- k8s
- CoreOS
- Hetzner
- Hosting
comments: []
---
> OKD4 wird langsam beliebt. Wer kein Geld für einen vollwertigen OpenShift Cluster hat, hat hier eine Chance auf den Service. Für VMs auf Basis von oVirt gibt es auf GitHub einige Projekte, die Installationen durchführen. Wie man aber einen Cluster auf echten Maschinen bei Hetzner aufbaut, muss man sich zusammensuchen.

Ich habe bis for ein paar Tagen einen OKD 3.11 Cluster als Single-Node-Cluster auf einem Node laufen gehabt. 64GB Hauptspeicher, 16 Cores. Da ich sowieso auf OKD 4 wechseln wollte und die Leistung am Ende war, habe ich mir jetzt drei Server mit jeweils 12 Cores und 64 GB Hauptspeicher geholt. Platten sind jeweils 500 GB enthalten. Als Loadblancer habe ich mir auf der Serverbörse zwei kleine Maschinen geschossen, die jeweils per haproxy auf den Cluster loadblancen. Außerdem werde ich dort ein paar weitere Dienste installieren, die nicht in Kubernetes laufen oder ich nicht dort haben will.

Außerdem habe ich mir temporär eine weitere Maschine als bootstrap-Maschine geholt - ich wollte eigentlich eine der kleineren Maschinen dazu nutzen, aber leider gab es da Netzwerkprobleme, da die entsprechende Netzwerkkarte von Fedora CoreOS nicht unterstützt wurde.

Ich hatte zwei Probleme:

1. Die Maschinen kamen alle als "localhost" hoch. Ich habe das dann so gelöst, dass ich per Ignition-File den einzelnen Hosts ihren Namen vorgegeben habe. Hat nur nichts gebracht. Also habe ich - sobald der Host hochkam - mit "hostnamectl" den Namen gesetzt und neu gestartet. Nachdem ich mit allen 3 Nodes durch war, musste ich nur noch per "oc delete node localhost" den blöden Zusatzeintrag loswerden.
2. Ich habe zwei Rechner im gleichen Subnetz bekommen. Dummerweise denken die dann, sie wären direkt über das Netz erreichbar, Hetzner verbietet aber natürlich direkte Kommunikation. Da ich mich mit Fedora CoreOS nicht ganz so gut auskenne und es nicht hinbekommen habe, diese Route zu ersetzen, habe ich mittels systemd-Timer einen Service aufgesetzt, der auf den Nodes alle 2 Sekunden die per DHCP gesetzte Route lösche und eine direkte Route zum Gateway setze.

Der Netzwerk-Workaround ist doof, aber die übliche dokumentierte Lösung über `<interface>.connection`-Datei hat bei mir nicht gegriffen.
