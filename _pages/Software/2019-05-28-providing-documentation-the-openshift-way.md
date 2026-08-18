---
layout: page
status: publish
published: true
title: Providing documentation the OpenShift way - Part 1
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 471
wordpress_url: https://www.lichti.de/?p=471
date: '2019-05-28 07:00:32 +0200'
date_gmt: '2019-05-28 05:00:32 +0200'
categories:
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
Documentation is one of the most hated part of the life of a developer. So the documentation is often the most neglected part of a project. At work I use Asciidoctor to write my customer documentation and it is quite acceptable. I loved LaTeX and Asciidoctor is an acceptable replacement for technical documentation - especially with the alternatives being google doc or word.

## Problem statement

Lets define the problem to solve first:

- A service/software needs the current documentation available on the internet.
- The documentation source should be saved in the same git repository as the software/service source code itself, so it is managed in the same lifecycle as the software itself.
- A static documentation should be generated and provided in form of HTML via HTTP (commonly known as "web server").
- It should be deployed within an OpenShift or OKD cluster.

## Asciidoctor as markup language for documentation

I don't want to advocate Asciidoctor, well in fact I do want to advocate for it. [Asciidoctor](https://asciidoctor.org/) is a nice way to write technical documentation as pure ascii (well, UTF-8) text and let compile it to nice output like PDF or HTML. But check other resources like the home page of Asciidoctor for the syntax and semantic of this text processing language. Believe me, it's really easy to write and read (even in unprocessed form).

The nice thing, the whole documentation may be checked in on the source code revisioning. Since we work with s2i-bilder I assume you use git. If that's a gitlab or public or private github or something like gogs, doesn't matter. If it is able to provide a remote git repo, that's fine.

## Writing documentation

Let's put the documentation in the directory /documentation of the git repo. There is an index.adoc file (the future landing page as index.html on the webserver). Perhaps you put the whole documentation into one single file (I talk about the output, the input may be split into more than one file included in the generation process). Or you have an hierarchy of documents.

How you structure your documentation is completely up to you. It only matters that there is a single index.adoc in the base directory of your documentation.

## Generating documentation

And that's it. I want to have a build configuration where I point to that git repository (the URI, the branch and the contextDir like /documentation) and the rest is done by software.

And how I solve this, is described in the upcoming articles:

- Part 2: [Creating the s2i builder with ASCIIDOC generation software included](https://www.lichti.de/2019/05/28/creating-the-s2i-builder-with-asciidoc-generation-software-included/)
- Part 3: [Creating the documentation site](https://www.lichti.de/2019/05/29/creating-the-documentation-site/)
- Part 4: [Bundling the components for OpenShift](https://www.lichti.de/2019/05/31/bundling-the-components-for-openshift/)
