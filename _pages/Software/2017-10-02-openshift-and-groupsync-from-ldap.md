---
layout: page
status: publish
published: true
title: Openshift and GroupSync from LDAP
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 331
wordpress_url: https://www.lichti.de/?p=331
date: '2017-10-02 12:00:35 +0200'
date_gmt: '2017-10-02 10:00:35 +0200'
categories:
- Softwareschnipsel
- English
tags:
- Active Directory
- AD
- Java
- LDAP
- OCP
- OpenShift
- Security
comments: []
---
OpenShift offers a variety of possible integrations into security providers. The integration is divided into authentication and authorization. Authentication is handled by one of the configurable IdentityProviders of OpenShift. While authorization is handled by importing groups into OpenShift. For importing groups the most used method is reading from an LDAP (or an Active Directory via its LDAP interface). OpenShift already has a [synchronization tool](https://docs.openshift.com/container-platform/3.6/install_config/syncing_groups_with_ldap.html) for this type of synchronization. And as long as that tool is sufficient, there are more reasons to stay with that tool than to replace it. But there are some situations where you need to replace it. And here the base software I written and published to github project [klenkes74/openshift-ldapsync](https://github.com/klenkes74/openshift-ldapsync).

## Reasons for using this software

Possible reasons (among others) not to use the official LDAP sync are:

- The SSL certificate of the LDAP could not be checked.
- The group structure is not supported. Mainly nested groups pose a (performance) problem.
- The linking element between user and groups (the username) is not exposed as single attribute on the LDAP.
- You don't have an LDAP server to sync with but other means of providing group authorization data.

Well, the last reason will result in a little bit more work since you have to replace the LDAP reading part of the software, but it's still doable and easy forward. The current software as provided on github will take care of the first three problems. But since your LDAP structure may vary, you will propably need to change the code.

## Integration into openShift

The LDAP sync will run as single pod within OpenShift. By using this approach we don't need to take care of system cron jobs or how to handle the case that the server this job is running on fails. OpenShift is good at managing that a pod is running, so it was a natural decision to give that job to OpenShift. In very restrictive environments you may have to define an egress router to be able to connect to the LDAP directory (or even the OpenShift API). But you will know that and there is quite good documentation for adding this [egress router](https://docs.openshift.com/container-platform/3.6/admin_guide/managing_pods.html#admin-guide-deploying-an-egress-router-pod-pods "Defining an egress router for OpenShift").

The README of the github project describes how to install the software and which parameter you need to set.

## Spring Boot

The software is a default spring-boot application. It leveraged the easy startup and scheduling functions of spring-boot to start and run the synchronization. The [application class](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/Application.java) is very straight forward and starts only the injected runner [SyncGroupsController](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/SyncGroupsController.java).

## Doing the work

The SyncGroupsController reads the groups from the LDAP and the OpenShift API into standardized maps (taking the OpenShift name as key and the [Group](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/groups/Group.java) as value). And then it runs all defined [group executors](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/GroupExecutor.java) and passing them the two maps. Currently only one executor is defined but you could add additional ones matching your needs (e.g. you could enrich already existing users with their email addresses or names if your authentication module does not deliver that data). It is really up to you.

But looking at the [SyncGroupEecutor](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/ocp/SyncGroupExecutor.java) (this is the class, the real work takes place) you see, it is really structured. It first creates selector sets for synched groups (groups that exist in LDAP and OpenShift), groups to add (groups that only exist in LDAP) and groups to delete (groups that only exist in OpenShift). As long as we have only one source for groups, this handling is simple and easy.

After having selected the different action items, the code creates a set of commands to run (these could be either [Create](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/ocp/actions/Create.java), [Update](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/ocp/actions/Update.java) or [Delete](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/ocp/actions/Delete.java)). Every command executes the changes for exactly one group. And this set of course is then executed. And that's it.

Well. But almost every LDAP looks different and one of the requirements was to handle it different. And you are right. That magic is hidden in the readers for OpenShift and LDAP (the things creating the maps with the Group). There are the readers mapping the data read from OpenShift or LDAP and providing the normalized Group object. Within these readers the converts convert from [OpenShift](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/ocp/OcpGroupReader.java) (since the OpenShift data does not change, the converter is within the same class) or LDAP (one for the [groups](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/ldap/LdapGroupConverter.java) and one for the [users](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/ldap/LdapUserConverter.java)). And here are the places to change the mappings. Feel free to adapt them to your needs.

## And the ugly rest ...

Well, and then there is the SSL part. I'm not proud of it. But especially the type of companies with big datacenters using OpenShift often have problems providing corect SSL certificates to their AD servers. So there is a small part of code in [LdapServer](https://github.com/klenkes74/openshift-ldapsync/blob/master/src/main/java/de/kaiserpfalzedv/ocp/groupsync/ldap/LdapServer.java) (lines 79 to 97), that copes with that problem:

```java
try {
 SSLContext ctx = SSLContext.getInstance("TLS");
 X509TrustManager tm = new X509TrustManager() {

public void checkClientTrusted(X509Certificate[] xcs, String string) throws CertificateException {
 }

public void checkServerTrusted(X509Certificate[] xcs, String string) throws CertificateException {
 }

public X509Certificate[] getAcceptedIssuers() {
 return null;
 }
 };
 ctx.init(null, new TrustManager[]{tm}, null);
 SSLContext.setDefault(ctx);
} catch (Exception ex) {
 ex.printStackTrace();
}
```
Of course there is some boilerplate code to read configuration from the system environment to be able to configure it the "OpenShift way". But you also could give every needed parameter via command line (to run it "adhoc" as a first import for example).

## Summary

Of course you should not use this software (or a derivate of it) when the good tested and easy configurable default LDAP sync mechanism works for you. Never add any security related component to your system if you don't have to. But If you need, then you have a nice basis to work on. Drop me an email or comment. And I'd love to receive pull requests to improve the software. Especially the tests are missing ...
