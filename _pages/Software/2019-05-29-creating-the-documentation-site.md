---
layout: page
status: publish
published: true
title: Creating the documentation site
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 495
wordpress_url: https://www.lichti.de/?p=495
date: '2019-05-29 03:24:34 +0200'
date_gmt: '2019-05-29 01:24:34 +0200'
categories:
- Softwareschnipsel
- OpenShift
- English
tags:
- OpenShift
- Origin
- OKD
- Kubernetes
- k8s
- asciidoctor
- documentation
comments: []
---
(**[Providing documentation the OpenShift way](https://www.lichti.de/2019/05/28/providing-documentation-the-openshift-way/)** – Part 3)

After preparing the software, now comes the ease part: writing the documentation. Ok, it's not the easiest part as every developer and system integrator knows.

But generating the documentation pod now is quite easy.

All you need is the git repo containing the documentation and a BuildConfig with the git coordinates of the documentation. The git coordinates are the git URI, the branch name and the contextDir of the documentation. If the documentation is in directory /documentation of your source repo, that's the contextDir.

The only thing the generator enforces is, that you need to have an index.adoc in the root of your contextDir. It will convert any file named \*.adoc inside the contextDir to HTML. And all files within the contextDir are copied to the output container. So images etc. may be transfered, too.

The result is a container containing a webserver and static data to be served. Just run it and you have a standalone static documentation site.

_The next part will take care of bundling everything in nice templates for easy use by the delivery projects._

_This article is part of the small series:_

- Part 1: [Providing documentation the OpenShift way](https://www.lichti.de/2019/05/28/providing-documentation-the-openshift-way/)
- Part 2: [Creating the s2i builder with ASCIIDOC generation software included](https://www.lichti.de/2019/05/28/creating-the-s2i-builder-with-asciidoc-generation-software-included/)
- Part 3: Creating the documentation site
- Part 4: [Bundling the components for OpenShift](https://www.lichti.de/2019/05/31/bundling-the-components-for-openshift/)
