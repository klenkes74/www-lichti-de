---
layout: page
status: publish
published: true
title: Bundling the components for OpenShift
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 502
wordpress_url: https://www.lichti.de/?p=502
date: '2019-05-31 08:19:16 +0200'
date_gmt: '2019-05-31 06:19:16 +0200'
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
<p><!-- wp:paragraph --></p>
<p>(<strong><a href="https://www.lichti.de/2019/05/28/providing-documentation-the-openshift-way/">Providing documentation the OpenShift way</a></strong> – Part 4)</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>The last two posts described the creation of the generic s2i builder and the concrete documentation site. This blog post now will put all the pieces together and provide a nice package to use on OpenShift or OKD. You get the templates, the build pipeline.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:more --><br />
<!--more--><br />
<!-- /wp:more --></p>
<p><!-- wp:heading --></p>
<h2>The Build Pipeline</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Looking at the generic asciidoc s2i builder first, we want to have a nice <a href="https://github.com/klenkes74/openshift-asciidoctor/blob/master/asciidoctor-s2i/Jenkinsfile">build pipeline</a>. The pipeline uses the built in jenkins of OpenShift to run the actual build, deploys a test docpod (using the asciidoc-s2i-builder documentation for these tests). If the docpod runs (which means that the readiness and liveness probes can access the index.html on the pod) the test succeeds and the generated image gets labeled "latest".</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>The pipeline uses the second build definition for the actual build. That one uses the dockerstrategy as pointed out in <a href="https://www.lichti.de/2019/05/28/creating-the-s2i-builder-with-asciidoc-generation-software-included/">part 2 of this blog series</a>. Result is the generic s2i builder with the label "test". Then the test deployment is run (using the documentation template we discuss in a few seconds). If that succeeds, the result is labeled "latest" and can be consumed by other projects.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:quote --></p>
<blockquote class="wp-block-quote"><p>If you want to grant access to the generic s2i builder to all projects, you need either grant this access to the building namespace you use for this pipeline or you create an imagestream in the namespace "openshift" that automatically pulls the "latest" tagged image from your building namespace. I provide a small sample in the github repository for this builder.</p>
<p><cite><a href="https://github.com/klenkes74/openshift-asciidoctor/blob/master/asciidoctor-s2i-openshift.yaml">https://github.com/klenkes74/openshift-asciidoctor/blob/master/asciidoctor-s2i-openshift.yaml</a></cite></p></blockquote>
<p><!-- /wp:quote --></p>
<p><!-- wp:paragraph --></p>
<p>Last not least you can create the webhook to start the pipeline. Calling "oc describe bc/nginx-asciidoctor-base-2.0.9" will provide the link to use for the webhook(s). Don't forget to replace &lt;secret> by the actual secret you chose (if you forgot, you can look it up in the WebUI, but that is another story to tell).</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>The Documentation Template</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>Now we can take a closer look to documentation.yaml - this is the template for the concrete documentation. It generates all the little objects needed.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>You will need it in the building environment of the asciidoc s2i builder (to get the tests working) and you need a slightly modified version in the namespace "openshift" for consumption by all other projects. The little change is, that the builder pod is not loaded locally. You need to set the namespace to "openshift" in the build config. And that's all. Now you can use the "browse catalog" to create your documentation pods needed for your project(s).</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p><em>This article is part of the small series:</em></p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>Part 1: <a href="https://www.lichti.de/2019/05/28/providing-documentation-the-openshift-way/">Providing documentation the OpenShift way</a></li>
<li>Part 2: <a href="https://www.lichti.de/2019/05/28/creating-the-s2i-builder-with-asciidoc-generation-software-included/">Creating the s2i builder with ASCIIDOC generation software included</a></li>
<li>Part 3: <a href="https://www.lichti.de/2019/05/29/creating-the-documentation-site/">Creating the documentation site</a></li>
<li>Part 4: Bundling the components for OpenShift</li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:paragraph --></p>
<p><!-- /wp:paragraph --></p>
