---
layout: page
status: publish
published: true
title: Blick in das JNDI ...
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 271
wordpress_url: https://www.lichti.de/?p=271
date: '2016-09-29 17:23:22 +0200'
date_gmt: '2016-09-29 15:23:22 +0200'
categories:
- Softwareschnipsel
- Deutsch
tags: []
comments: []
---
Manchmal muss man einfach in einen vorhanden JNDI reinschauen, um herauszufinden, was dort eigentlich auf Entdeckung schlummert. Die folgende Klasse liefert einen formatierten Baum aus den JNDI-Eintragungen.

```java
package de.kaiserpfalzedv.office.common.jndi;

import javax.naming.InitialContext;
import javax.naming.NameClassPair;
import javax.naming.NamingEnumeration;
import javax.naming.NamingException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * A small utlitiy class to print the JNDI tree.
 *
 * @author klenkes {@literal <rlichti@kaiserpfalz-edv.de>}
 * @version 1.0.0
 * @since 2016-09-29
 */
public class JndiWalker {
 private static final Logger LOG = LoggerFactory.getLogger(JndiWalker.class);

/**
 * Walks the {@link InitialContext} with the entry point given and returns a string containing all sub entries.
 *
 * @param context The initial context to be printed.
 * @param entryPoint The node to start with.
 * @return A string containing the tree of the initial context (nodes including the leaves).
 * @throws NamingException If the lookup failes for some unknown reason.
 */
 public String walk(final InitialContext context, final String entryPoint) throws NamingException {
 StringBuffer sb = new StringBuffer(" JNDI tree for '").append(entryPoint).append("': ");

LOG.trace("JNDI-Walker activated for entry point '{}' on: {}", entryPoint, context);
 walk(sb, context, entryPoint, 0);

return sb.toString();
 }

private void walk(
 StringBuffer sb,
 final InitialContext context,
 final String name,
 final int level
 ) throws NamingException {
 NamingEnumeration ne;
 try {
 ne = context.list(name);
 } catch (NamingException e) {
 LOG.trace("Reached leaf of JNDI: {}", name);
 return;
 }

while (ne.hasMoreElements()) {
 NameClassPair ncp = (NameClassPair) ne.nextElement();

printLevel(sb, level);

sb
 .append(ncp.getName())
 .append(" (").append(ncp.getClassName())
 .append(", ").append(getNameInNamespace(ncp))
 .append(", ").append(getRelativeFlag(ncp))
 .append(")");

walk(sb, context, name + "/" + ncp.getName(), level + 4);
 }
 }

private void printLevel(StringBuffer sb, int level) {
 sb.append("\n");
 for (int i=0; i < level; i++) {
 sb.append(" ");
 }
 }

private String getNameInNamespace(NameClassPair ncp) {
 String nameInNamespace;
 try {
 nameInNamespace = ncp.getNameInNamespace();
 } catch (UnsupportedOperationException e) {
 nameInNamespace = "not-supported";
 }
 return nameInNamespace;
 }

private String getRelativeFlag(NameClassPair ncp) {
 String isRelative;
 try {
 isRelative = ncp.isRelative() ? "relative" : "fixed";
 } catch (UnsupportedOperationException e) {
 isRelative = "not-supported";
 }
 return isRelative;
 }
}
```
Die Basis für diese kleine Hilfsklasse war ein [Blogeintrag](http://andersnorgaard.blogspot.de/2005/08/piece-of-java-code-to-walk-jndi-tree.html) von [Anders Rudklaer Norgaard](https://plus.google.com/108162764738103442458). Ich habe sie nur Servlet-unabhängig gemacht.
