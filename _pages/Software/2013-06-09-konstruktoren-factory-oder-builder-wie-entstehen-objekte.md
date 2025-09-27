---
layout: page
status: publish
published: true
title: Konstruktoren, Factory oder Builder? Wie entstehen Objekte?
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 149
wordpress_url: https://www.lichti.de/?p=149
date: '2013-06-09 07:07:12 +0200'
date_gmt: '2013-06-09 07:07:12 +0200'
categories:
- Softwareschnipsel
- Deutsch
tags: []
comments: []
---
<p>Im Netz wurde schon sehr viel geschrieben, wie man Objekte erschaffen kann. Es ist eines der Grundprobleme bei der Softwareentwicklung und wie bei vielen wichtigen Entscheidungen gibt es eine klare Antwort: Es kommt auf die Situation an.</p>
<p><!--more--></p>
<p>Die einfachste Fassung ist es, den Konstruktor direkt aufzurufen. Das funktioniert immer, wird aber ab einer bestimmten Anzahl von Paramtern leicht unübersichtlich, vor allem wenn viele diese Parameter vom gleichen Typ sind. Das Typenproblem kann aber auch schon bei sehr wenigen Parametern auftreten:</p>
<p>[java autolinks="false" collapse="false" firstline="1" gutter="true" highlight="11,20-27" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
package de.kaiserpfalzEdv.blog.creation;</p>
<p>import static com.google.common.base.Preconditions.checkArgument;<br />
import static org.apache.commons.lang3.StringUtils.isNotBlank;</p>
<p>public class ConstructorDemo {<br />
    ...</p>
<p>    public demo() {<br />
        ...<br />
        ValueObject object = new ValueObject(&quot;Meier&quot;, &quot;Hermann&quot;);<br />
        ...<br />
    }<br />
}</p>
<p>class ValueObject {<br />
    private String familyName;<br />
    private String givenName;</p>
<p>    public ValueObject(final String familyName, final String givenName) {<br />
        checkArgument(isNotBlank(familyName));<br />
        checkArgument(isNotBlank(givenName));</p>
<p>        this.familyName = familyName;<br />
        this.givenName = givenName;<br />
    }</p>
<p>    public String getFamilyName() {<br />
        return familyName;<br />
    }</p>
<p>    public String getGivenName() {<br />
        return givenName;<br />
    }<br />
}<br />
[/java]</p>
<p>Bei diesem Bespiel könnte es nach einiger Zeit (oder bei einem anderen Entwickler) fraglich sein, ob man zuerst den Nachnamen oder den Vornamen übergeben muss. Noch schlimmer wird es, wenn es vier oder fünf Parameter werden und diese eventuell sogar den gleichen Typ haben - da kann selbst die beste IDE nicht mehr helfen und es wird fast automatisch irgendwann zu Verdrehern kommen.</p>
<p>Das <a title="Builder-Pattern" href="http://de.wikipedia.org/wiki/Erbauer_%28Entwurfsmuster%29" target="_blank" rel="noopener noreferrer">Builder Pattern</a> (deutsch: Erbauer) hilft es, diese Klippe zu umschiffen. Hier hilft eine weitere Klasse, der sogenannte Builder, das Objekt zu erschaffen:</p>
<p>[java autolinks="false" collapse="false" firstline="1" gutter="true" highlight="12,35-57" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
package de.kaiserpfalzEdv.blog.creation;</p>
<p>import java.lang.IllegalStateException;<br />
import static org.apache.commons.lang3.StringUtils.isNotBlank;</p>
<p>public class BuilderDemo {<br />
    ...</p>
<p>    public demo() {<br />
        ...<br />
        ValueObject object = new ValueObject.Builder().withFamilyName(&quot;Meier&quot;).withGivenName(&quot;Hermann&quot;).build();<br />
        ...<br />
    }<br />
}</p>
<p>class ValueObject {<br />
    private String familyName;<br />
    private String givenName;</p>
<p>    private ValueObject() {}</p>
<p>    private boolean isValid() {<br />
        return isNotBlank(familyName) &amp;&amp; isNotBlank(givenName);<br />
    }</p>
<p>    public String getFamilyName() {<br />
        return familyName;<br />
    }</p>
<p>    public String getGivenName() {<br />
        return givenName;<br />
    }</p>
<p>    public static class Builder() {<br />
        private ValueObject buildObject = new DataClass();</p>
<p>        public ValueObject build() {<br />
            if (buildObject.isValid()) {<br />
                return buildObject;<br />
            } else {<br />
                throw new IllegalStateException(&quot;Sorry, DataObject can not be created with the given data.&quot;);<br />
            }<br />
        }</p>
<p>        public Builder withFamilyName(final String name) {<br />
            buildObject.familyName = name;</p>
<p>            return this;<br />
        }</p>
<p>        public Builder withGivenName(final String name) {<br />
            buildObject.givenName = name;</p>
<p>            return this;<br />
        }<br />
    }<br />
}<br />
[/java]</p>
<p>Jetzt können die Paramter nur noch verwechselt werden, wenn der Entwickler nicht weiß, was "familyName" und was "givenName" bedeutet. Aber dann hat man ein ganz anderes Problem. Wie man aber am Code sieht, muss man sich ein paar Gedanken machen: der eigentliche Konstruktor ist nun private, kann also nicht mehr direkt aufgerufen werden.</p>
<p>Das funktioniert, wenn der Builder wie hier eine innere Klasse zum ValueObject darstellt. Bei so kleinen Objekten wie diesem hier geht das auch noch, kann aber unübersichtlich werden.</p>
<p>Bleibt einem so also nicht die Wahl und man muss den Builder als eigene Klasse bauen, dann könnte man das eigentliche Value-Objekt package-local definieren (also ohne "public") und den Builder ins gleiche Package stecken. Der Konstruktor des Value-Objektes müsste demnach dann auch package-lokal sein, was die enge Kapselung leider wieder etwas öffnet. Auch müsste man die Variablen des Value-Objektes package-lokal definieren (oder entsprechende Setter, die wie man oben sieht hier vollständig fehlen).</p>
<p>Im Builder kann man aber auch noch etwas anderes verstecken: Werden oft Objekte mit den gleichen Daten benötigt, so könnten diese in einen Cache gelegt werden und dann diese wieder ausgegeben werden. Da es sich um ValueObjekte handelt, deren Status (hier: Vor- und Nachname) sich nicht mehr ändern kann, kann auch die Software in Wirklichkeit mit einem einzelnen Objekt arbeiten und braucht nicht immer ein neues Objekt. Es kann also dadurch optimiert werden.</p>
<p>Was mache ich aber, wenn die die Möglichkeit des Wiederverwendens eines Objekts nutzen will, aber ein Builder mit Kanonen auf Spatzen geschossen wäre (zum Beispiel, wenn ich nur einen Parameter habe)?</p>
<p>Dann kann ich eine <a title="Factory" href="http://en.wikipedia.org/wiki/Factory_%28software_concept%29" target="_blank" rel="noopener noreferrer">Factory</a> nutzen.</p>
<p>[java autolinks="false" collapse="false" firstline="1" gutter="true" highlight="11,34-39" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
package de.kaiserpfalzEdv.blog.creation;</p>
<p>import static com.google.common.base.Preconditions.checkArgument;<br />
import static org.apache.commons.lang3.StringUtils.isNotBlank;</p>
<p>public class ConstructorDemo {<br />
    ...</p>
<p>    public demo() {<br />
        ...<br />
        ValueObject object = ValueObject.Factory.getInstance(&quot;Meier&quot;, &quot;Hermann&quot;);<br />
        ...<br />
    }<br />
}</p>
<p>class ValueObject {<br />
    private String familyName;<br />
    private String givenName;</p>
<p>    private ValueObject(final String familyName, final String givenName) {<br />
        this.familyName = familyName;<br />
        this.givenName = givenName;<br />
    }</p>
<p>    public String getFamilyName() {<br />
        return familyName;<br />
    }</p>
<p>    public String getGivenName() {<br />
        return givenName;<br />
    }</p>
<p>    public static class Factory {<br />
        public static ValueObject getInstance(final String familyName, final String givenName) {<br />
            checkArgument(isNotBlank(familyName));<br />
            checkArgument(isNotBlank(givenName));</p>
<p>            return new ValueObject(familyName, givenName);<br />
        }<br />
    }<br />
}<br />
[/java]</p>
<p>Die Factory unterscheidet sich von der <a title="Factory-Method" href="http://en.wikipedia.org/wiki/Factory_method_pattern" target="_blank" rel="noopener noreferrer">Factory-Methode</a> dadurch, dass sie eine eigene Klasse ist, die eine oder mehrere Factory-Methoden in sich vereint, während die Factory-Methode eine statische Methode in der eigentlichen Klasse darstellt:</p>
<p>[java autolinks="false" collapse="false" firstline="1" gutter="true" highlight="11,25-30" htmlscript="false" light="false" padlinenumbers="false" smarttabs="true" tabsize="2" toolbar="true"]<br />
package de.kaiserpfalzEdv.blog.creation;</p>
<p>import static com.google.common.base.Preconditions.checkArgument;<br />
import static org.apache.commons.lang3.StringUtils.isNotBlank;</p>
<p>public class ConstructorDemo {<br />
    ...</p>
<p>    public demo() {<br />
        ...<br />
        ValueObject object = ValueObject.getInstance(&quot;Meier&quot;, &quot;Hermann&quot;);<br />
        ...<br />
    }<br />
}</p>
<p>class ValueObject {<br />
    private String familyName;<br />
    private String givenName;</p>
<p>    private ValueObject(final String familyName, final String givenName) {<br />
        this.familyName = familyName;<br />
        this.givenName = givenName;<br />
    }</p>
<p>    public static ValueObject getInstance(final String familyName, final String givenName) {<br />
        checkArgument(isNotBlank(familyName));<br />
        checkArgument(isNotBlank(givenName));</p>
<p>        return new ValueObject(familyName, givenName);<br />
    }</p>
<p>    public String getFamilyName() {<br />
        return familyName;<br />
    }</p>
<p>    public String getGivenName() {<br />
        return givenName;<br />
    }<br />
}<br />
[/java]</p>
<p>Damit haben wir nun alle vier Möglichkeiten gesehen. Welche passt am besten? Und die Antwort hatten wir schon zu Anfang: Es kommt auf die Situation an. Eventuell kann die folgende Kurzliste helfen ...</p>
<ol>
<li>Das Objekt hat nur wenige (1-3) Parameter, die auch nicht leicht zu verwechseln sind. Außerdem soll es jedesmal neu geschaffen werden. Ein typischer Fall für den Konstruktor-Aufruf per "new Object(...)".</li>
<li>Das Objekt hat nur wenige (1-3) Parameter, die auch nicht leicht zu verwechseln sind. Es soll eventuell gecacht werden. Ein typischer Fall für die Factory-Method innerhalb des Objekts.</li>
<li>Das Objekt hat nur wenige (1-3) Parameter, die auch nicht leicht zu wechseln sind. Es soll eventuell gecacht werden oder gar eine Subklasse generiert werden (abhängig von den Parametern). Ein typischer Fall für die Factory.</li>
<li>Das Objekt hat viele Paramter oder wenige Paramter, die leicht zu verwechseln sind. Hier kann der Builder echt helfen ...</li>
</ol>
