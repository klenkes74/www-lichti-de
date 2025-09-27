---
layout: page
status: publish
published: true
title: OpenShift or Kubernetes cluster configuration catalogue
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 564
wordpress_url: https://www.lichti.de/?p=564
date: '2020-04-07 23:44:31 +0200'
date_gmt: '2020-04-07 21:44:31 +0200'
categories:
- Softwareschnipsel
- OpenShift
- English
tags:
- OCP
- Kubernetes
- k8s
comments: []
---
<p><!-- wp:paragraph --></p>
<p>Kubernetes is not kubernetes. Every cluster is configured in a special way and offers additional features. Some of them are build in the distribution, like OpenShift contains for example a default ingress service (the router) - others are provided by the team maintaining the cluster. Or the maintaining team of the cluster decided not to provide certain features of k8s or the distribution used.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>How do you communicate the feature set you provide to your customers. For a single cluster and a small group of users it's easy: you explain it to your users. But the bigger the cluster grows and the more users you have, you find out: this does not scale. And adding multiple clusters in different versions, it becomes a mess.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>But you could use a k8s feature to build a catalogue of features of the current cluster. You define the feature sets and add the installed features to the cluster and your users may query the cluster about the supported features of the cluster they want to use.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>The k8s feature I'm talking about is the custom resource. Just create a custom resource containing the information you want to provide and add the features to the catalogue. Then the catalogue can be queried like this:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:preformatted --></p>
<pre class="wp-block-preformatted">$ oc get ift
NAME                 GROUP          VERSION        AGE       DOCUMENTATION
features-catalogue   cluster-info   1.0.0-alpha1   1d        https://github.com/klenkes74/k8s-installed-features-catalogue/
$</pre>
<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p>I don't want to double the information, so I point for the implementation to my github repo containing an implementation of this idea: <a href="https://github.com/klenkes74/k8s-installed-features-catalogue">https://github.com/klenkes74/k8s-installed-features-catalogue</a>. Please comment and write your opinion of such a catalogue.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph {"fontSize":"small"} --></p>
<p class="has-small-font-size">Titelbild: <a href="https://www.rgbstock.com/photo/mD1OYUK/old+phonebook">old phonebook</a> (<a href="https://www.rgbstock.com/user/lusi">lusi</a>@<a href="https://www.rgbstock.com">rgbstock</a>, <a href="https://www.rgbstock.com/license">RGBStock Lizenz</a>)</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p><!-- /wp:paragraph --></p>
