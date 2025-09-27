---
layout: page
status: publish
published: true
title: Creating the s2i builder with ASCIIDOC generation software included
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 479
wordpress_url: https://www.lichti.de/?p=479
date: '2019-05-28 07:10:32 +0200'
date_gmt: '2019-05-28 05:10:32 +0200'
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
<p>(<strong><a href="https://www.lichti.de/2019/05/28/providing-documentation-the-openshift-way/">Providing documentation the OpenShift way</a></strong> - Part 2)</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>For building the documentation pod, we need two components: the asciidoc html generator and the webserver for delivering the static pages later. There are several base containers published containing either the asciidoc generator or the webserver. I liked the converter published on <a href="https://github.com/asciidoctor/docker-asciidoctor">https://github.com/asciidoctor/docker-asciidoctor</a>. But that is only the generator part. On the other hand there are default bsic containers like httpd or nginx containing the web server part. Or, as third option you could use a ruby s2i builder as starting point and add both, Asciidoc <strong>and</strong> the web server later.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:more --><br />
<!--more--><br />
<!-- /wp:more --></p>
<p><!-- wp:paragraph --></p>
<p>I decided to use the webserver base container for nginx as basis and install the needed software (ruby, python, a lot of GEMs and Python packages) via a docker file. A Dockerfile in contrast to changing s2i-builder scripts was needed since we need to install as UID 0 and switch later to the generic UID 1001 for OpenShift and s2i only runs on the dynamic generated UID. So the base image is an s2i builder image but we don't use the s2i technology but prepare a new s2i builder by running a <a href="https://raw.githubusercontent.com/klenkes74/openshift-asciidoctor/master/asciidoctor-s2i/Dockerfile">Dockerfile</a>:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:quote --></p>
<blockquote class="wp-block-quote"><p><code>FROM openshift/nginx:1.14<br> <br>EXPOSE 8080 <br>EXPOSE 8443 <br> <br>USER 0 <br> <br>RUN yum install -y centos-release-scl-rh &amp;&amp; yum install -y bash curl ca-certificates make java-1.8.0-openjdk-headless findutils diffutils patch inotify-tools rh-ruby23 rh-ruby23-devel rh-ruby23-ruby-devel rh-ruby23-scldevel rh-ruby23-build baekmuk-ttf-fonts-common  graphviz-devel graphviz libxml2-devel gcc python27-python-pip rh-ror42-rubygem-nokogiri &amp;&amp; yum clean all -y <br> <br>RUN scl enable rh-ruby23 "gem install --no-document asciidoctor:${ASCIIDOCTOR_VERSION} asciidoctor-pdf:${ASCIIDOCTOR_PDF_VERSION} asciidoctor-confluence:${ASCIIDOCTOR_CONFLUENCE_VERSION} asciidoctor-diagram:${ASCIIDOCTOR_DIAGRAM_VERSION} ruby-enum asciimath asciidoctor-revealjs:${ASCIIDOCTOR_REVEALJS_VERSION} asciidoctor-bibliography:${ASCIIDOCTOR_BIBLIOGRAPHY_VERSION} coderay epubcheck:3.0.1 haml kindlegen:3.0.3 pygments.rb rake rouge slim thread_safe tilt graphviz" <br> <br>RUN scl enable python27 "pip install --upgrade pip" &amp;&amp; scl enable python27 "pip install --no-cache-dir actdiag blockdiag[pdf] nwdiag Pygments seqdiag" &amp;&amp; rm -rf .cache<br> <br>RUN sed -i -e 's/set -e/set -e\n\ \n\ cp -a \<br>/tmp\/src\/* .\n\ \n\ if [ !  -f index.adoc ] ; then\n\<br>     echo "===================================================="\n\<br>     echo "   No index.adoc!!!!"\n\     echo "===================================================="\n\<br>     exit 1\n\ <br>fi\n\ <br>\n\<br>for adoc in `find . -iname "*.adoc"` ; do\n\<br>     echo "---&gt; Converting: ${adoc} ..."\n\<br>     scl enable python27 rh-ruby23 "asciidoctor -b xhtml5 --safe -n -v -t -r asciidoctor-diagram \"${adoc}\""\n\<br>     scl enable python27 rh-ruby23 "asciidoctor-pdf -b xhtml5 --safe -n -v -t -r asciidoctor-diagram \"${adoc}\""\n\<br>done\n\ /' ${STI_SCRIPTS_PATH}/assemble<br> <br>USER 1001<br> <br>CMD ${STI_SCRIPTS_PATH}/usage</code></p>
<p><cite>Dockerfile von <a href="https://github.com/klenkes74/openshift-asciidoctor/blob/master/asciidoctor-s2i/Dockerfile">GitHub (klenkes74)</a></cite></p></blockquote>
<p><!-- /wp:quote --></p>
<p><!-- wp:paragraph --></p>
<p>On basis of the nginx s2i container we install some software and change the existing s2i assemble script via a sed call to run asciidoctor and asciidoctor-pdf for the generation of the html and pdf version of the documentation. The versions of some of the package must be specified as environment arguments during the build.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>With this container the base is ready for lots of documentation containers. How we integrate it into OpenShift for build and usage is discussed in part 4 of this mini series.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p><em>This article is part of the small series:</em></p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>Part 1: <a href="https://www.lichti.de/2019/05/28/providing-documentation-the-openshift-way/">Providing documentation the OpenShift way</a></li>
<li>Part 2: Creating the s2i builder with ASCIIDOC generation software included</li>
<li>Part 3: <a href="https://www.lichti.de/2019/05/29/creating-the-documentation-site/">Creating the documentation site</a></li>
<li>Part 4: <a href="https://www.lichti.de/2019/05/31/bundling-the-components-for-openshift/">Bundling the components for OpenShift</a></li>
</ul>
<p><!-- /wp:list --></p>
