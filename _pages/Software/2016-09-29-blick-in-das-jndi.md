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
<p>Manchmal muss man einfach in einen vorhanden JNDI reinschauen, um herauszufinden, was dort eigentlich auf Entdeckung schlummert. Die folgende Klasse liefert einen formatierten Baum aus den JNDI-Eintragungen.</p>
<p><!--more--></p>
<p>[java autolinks="false" collapse="false" firstline="1" gutter="true" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
package de.kaiserpfalzedv.office.common.jndi;</p>
<p>import javax.naming.InitialContext;<br />
import javax.naming.NameClassPair;<br />
import javax.naming.NamingEnumeration;<br />
import javax.naming.NamingException;</p>
<p>import org.slf4j.Logger;<br />
import org.slf4j.LoggerFactory;</p>
<p>/**<br />
 * A small utlitiy class to print the JNDI tree.<br />
 *<br />
 * @author klenkes {@literal &lt;rlichti@kaiserpfalz-edv.de&gt;}<br />
 * @version 1.0.0<br />
 * @since 2016-09-29<br />
 */<br />
public class JndiWalker {<br />
  private static final Logger LOG = LoggerFactory.getLogger(JndiWalker.class);</p>
<p>  /**<br />
    * Walks the {@link InitialContext} with the entry point given and returns a string containing all sub entries.<br />
    *<br />
    * @param context The initial context to be printed.<br />
    * @param entryPoint The node to start with.<br />
    * @return A string containing the tree of the initial context (nodes including the leaves).<br />
    * @throws NamingException If the lookup failes for some unknown reason.<br />
    */<br />
  public String walk(final InitialContext context, final String entryPoint) throws NamingException {<br />
    StringBuffer sb = new StringBuffer(&quot; JNDI tree for '&quot;).append(entryPoint).append(&quot;': &quot;);</p>
<p>    LOG.trace(&quot;JNDI-Walker activated for entry point '{}' on: {}&quot;, entryPoint, context);<br />
    walk(sb, context, entryPoint, 0);</p>
<p>    return sb.toString();<br />
  }</p>
<p>  private void walk(<br />
          StringBuffer sb,<br />
          final InitialContext context,<br />
          final String name,<br />
          final int level<br />
  ) throws NamingException {<br />
    NamingEnumeration ne;<br />
    try {<br />
      ne = context.list(name);<br />
    } catch (NamingException e) {<br />
      LOG.trace(&quot;Reached leaf of JNDI: {}&quot;, name);<br />
      return;<br />
    }</p>
<p>    while (ne.hasMoreElements()) {<br />
      NameClassPair ncp = (NameClassPair) ne.nextElement();</p>
<p>      printLevel(sb, level);</p>
<p>      sb<br />
          .append(ncp.getName())<br />
          .append(&quot; (&quot;).append(ncp.getClassName())<br />
          .append(&quot;, &quot;).append(getNameInNamespace(ncp))<br />
          .append(&quot;, &quot;).append(getRelativeFlag(ncp))<br />
          .append(&quot;)&quot;);</p>
<p>      walk(sb, context, name + &quot;/&quot; + ncp.getName(), level + 4);<br />
    }<br />
  }</p>
<p>  private void printLevel(StringBuffer sb, int level) {<br />
    sb.append(&quot;\n&quot;);<br />
    for (int i=0; i &lt; level; i++) {<br />
      sb.append(&quot; &quot;);<br />
    }<br />
  }</p>
<p>  private String getNameInNamespace(NameClassPair ncp) {<br />
    String nameInNamespace;<br />
    try {<br />
      nameInNamespace = ncp.getNameInNamespace();<br />
    } catch (UnsupportedOperationException e) {<br />
      nameInNamespace = &quot;not-supported&quot;;<br />
    }<br />
    return nameInNamespace;<br />
  }</p>
<p>  private String getRelativeFlag(NameClassPair ncp) {<br />
    String isRelative;<br />
    try {<br />
      isRelative = ncp.isRelative() ? &quot;relative&quot; : &quot;fixed&quot;;<br />
    } catch (UnsupportedOperationException e) {<br />
      isRelative = &quot;not-supported&quot;;<br />
    }<br />
    return isRelative;<br />
  }<br />
}<br />
[/java]</p>
<p>Die Basis für diese kleine Hilfsklasse war ein <a href="http://andersnorgaard.blogspot.de/2005/08/piece-of-java-code-to-walk-jndi-tree.html">Blogeintrag</a> von <a href="https://plus.google.com/108162764738103442458">Anders Rudklaer Norgaard</a>. Ich habe sie nur Servlet-unabhängig gemacht.</p>
