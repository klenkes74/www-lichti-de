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
<p><!-- wp:paragraph --></p>
<p>Documentation is one of the most hated part of the life of a developer. So the documentation is often the most neglected part of a project. At work I use Asciidoctor to write my customer documentation and it is quite acceptable. I loved LaTeX and Asciidoctor is an acceptable replacement for technical documentation - especially with the alternatives being google doc or word.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:more --><br />
<!--more--><br />
<!-- /wp:more --></p>
<p><!-- wp:heading --></p>
<h2>Problem statement</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Lets define the problem to solve first:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>A service/software needs the current documentation available on the internet.</li>
<li>The documentation source should be saved in the same git repository as the software/service source code itself, so it is managed in the same lifecycle as the software itself.</li>
<li>A static documentation should be generated and provided in form of HTML via HTTP (commonly known as "web server").</li>
<li>It should be deployed within an OpenShift or OKD cluster.</li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:heading --></p>
<h2>Asciidoctor as markup language for documentation</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>I don't want to advocate Asciidoctor, well in fact I do want to advocate for it. <a href="https://asciidoctor.org/">Asciidoctor</a> is a nice way to write technical documentation as pure ascii (well, UTF-8) text and let compile it to nice output like PDF or HTML. But check other resources like the home page of Asciidoctor for the syntax and semantic of this text processing language. Believe me, it's really easy to write and read (even in unprocessed form).</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>The nice thing, the whole documentation may be checked in on the source code revisioning. Since we work with s2i-bilder I assume you use git. If that's a gitlab or public or private github or something like gogs, doesn't matter. If it is able to provide a remote git repo, that's fine.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>Writing documentation</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Let's put the documentation in the directory /documentation of the git repo. There is an index.adoc file (the future landing page as index.html on the webserver). Perhaps you put the whole documentation into one single file (I talk about the output, the input may be split into more than one file included in the generation process). Or you have an hierarchy of documents.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>How you structure your documentation is completely up to you. It only matters that there is a single index.adoc in the base directory of your documentation.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>Generating documentation</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>And that's it. I want to have a build configuration where I point to that git repository (the URI, the branch and the contextDir like /documentation) and the rest is done by software.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>And how I solve this, is described in the upcoming articles:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>Part 2: <a href="https://www.lichti.de/2019/05/28/creating-the-s2i-builder-with-asciidoc-generation-software-included/">Creating the s2i builder with ASCIIDOC generation software included</a></li>
<li>Part 3: <a href="https://www.lichti.de/2019/05/29/creating-the-documentation-site/">Creating the documentation site</a></li>
<li>Part 4: <a href="https://www.lichti.de/2019/05/31/bundling-the-components-for-openshift/">Bundling the components for OpenShift</a></li>
</ul>
<p><!-- /wp:list --></p>
