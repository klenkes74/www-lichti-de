---
layout: page
status: publish
published: true
title: Buildsystem via Vagrant-Box
author:
  display_name: klenkes74
  login: klenkes74
  email: roland@lichti.de
  url: ''
author_login: klenkes74
author_email: roland@lichti.de
wordpress_id: 278
wordpress_url: https://www.lichti.de/?p=278
date: '2016-11-28 11:33:13 +0100'
date_gmt: '2016-11-28 10:33:13 +0100'
categories:
- Softwareschnipsel
- Deutsch
tags:
- build
- ci
- configurationmanagement
comments: []
---
<p>In manchen Java-Projekten hat man das Problem, dass Entwickler unter Windows, Linux und MacOS arbeiten. Damit stellt schon das Build-Tooling ein Problem dar. Obwohl moderne Build-Systeme meistens in der JVM laufen und damit eigentlich betriebssystemunabhängig sein sollten, stößt man oft an das Problem, dass Pfadangaben leider nicht so unabhängig sind.</p>
<p>Um trotzdem lokal bauen zu können, wäre ein System schön, das wie ein zentraler Build-Server fungiert. Aber halt lokal. Und hier kommt Vagrant ins Spiel ...</p>
<p><!--more--></p>
<p>In den meisten Fällen dürfte der Buildserver auf einem Linuxsystem laufen, die sind billig und schnell aufgesetzt. Meistens ist das die Qualifikation dafür, da das Management für solchen "Overhead" kein Geld ausgeben will. Daher betrachte ich hier nur diesen Fall[ref name="dislike-windows"]Außerdem mag ich Windows und MacOS als Server nicht.[/ref].</p>
<p>Was braucht man alles?</p>
<ol>
<li>Eine Virtualisierungsumgebung (z.B. VMWare oder VirtualBox).</li>
<li>Vagrant zum Erzeugen der Virtual Machine.</li>
<li>Zeit, Lust und Durchhaltevermögen zum Erstellen des Vagrantfiles für die lokale Box.</li>
</ol>
<p>Ich benutze hier VirtualBox auf einem MacBook Pro. Zusammen mit der VirtualBox ist das eine nette Kombination. Vagrant gibt es auf als Download auf <a href="https://www.vagrantup.com/downloads.html">https://www.vagrantup.com/downloads.html</a>[ref name="vagrant-version"]Die aktuelle Version 1.8.7 für den Mac sind ca. 70 MB, je nach Netzwerk ist es also sofort da oder dauert etwas.[/ref]. In Version 1.8.7 ist wohl  das gelieferte curl defekt und muss gelöscht werden:</p>
<p>[code lang="bash" toolbar="true" title="Entfernen des defekten curl"]<br />
hermes:vagrant klenkes$ rm /opt/vagrant/embedded/bin/curl<br />
hermes:vagrant klenkes$<br />
[/code]</p>
<p><span class="s1">Damit  funktionieren die Box-Downloads[ref]siehe <a href="https://github.com/twobitcircus/rpi-build-and-boot/issues/25">https://github.com/twobitcircus/rpi-build-and-boot/issues/25</a>[/ref]. Danach </span><span class="s1">kann man per </span></p>
<p>[code lang="bash" toolbar="true" title="Herunterladen der Box mit Centos/7"]<br />
hermes:vagrant klenkes$ vagrant box add centos/7<br />
==&gt; box: Loading metadata for box 'centos/7'<br />
    box: URL: https://atlas.hashicorp.com/centos/7<br />
This box can work with multiple providers! The providers that it<br />
can work with are listed below. Please review the list and choose<br />
the provider you will be working with.</p>
<p>1) libvirt<br />
2) virtualbox<br />
3) vmware_desktop<br />
4) vmware_fusion</p>
<p>Enter your choice: 2<br />
==&gt; box: Adding box 'centos/7' (v1610.01) for provider: virtualbox<br />
    box: Downloading: https://atlas.hashicorp.com/centos/boxes/7/versions/1610.01/providers/virtualbox.box<br />
==&gt; box: Successfully added box 'centos/7' (v1610.01) for 'virtualbox'!<br />
hermes:vagrant klenkes$<br />
[/code]</p>
<p>eine Boxdefinition herunterladen. Ein Update könnte man mit</p>
<p>[code lang="bash" toolbar="true" title="Updaten einer Box-Definition"]<br />
hermes:vagrant klenkes$ vagrant box update --box centos/7<br />
Checking for updates to 'centos/7'<br />
Latest installed version: 1610.01<br />
Version constraints: &amp;gt; 1610.01<br />
Provider: virtualbox<br />
Box 'centos/7' (v1610.01) is running the latest version.<br />
hermes:vagrant klenkes$<br />
[/code]</p>
<p>ausgeführt werden.</p>
<p>Und mit</p>
<p>[code lang="bash" toolbar="true" title="Ausführen einer Vagrant-Box"]<br />
hermes:vagrant klenkes$ vagrant up<br />
Bringing machine 'default' up with 'virtualbox' provider...<br />
==&gt; default: Clearing any previously set forwarded ports...<br />
==&gt; default: Clearing any previously set network interfaces...<br />
==&gt; default: Preparing network interfaces based on configuration...<br />
    default: Adapter 1: nat<br />
    default: Adapter 2: hostonly<br />
==&gt; default: Forwarding ports...<br />
    default: 80 (guest) =&gt; 8080 (host) (adapter 1)<br />
    default: 22 (guest) =&gt; 2222 (host) (adapter 1)<br />
==&gt; default: Running 'pre-boot' VM customizations...<br />
==&gt; default: Booting VM...<br />
==&gt; default: Waiting for machine to boot. This may take a few minutes...<br />
    default: SSH address: 127.0.0.1:2222<br />
    default: SSH username: vagrant<br />
    default: SSH auth method: private key<br />
    default: Warning: Remote connection disconnect. Retrying...<br />
==&gt; default: Machine booted and ready!<br />
==&gt; default: Checking for guest additions in VM...<br />
    default: No guest additions were detected on the base box for this VM! Guest<br />
    default: additions are required for forwarded ports, shared folders, host only<br />
    default: networking, and more. If SSH fails on this machine, please install<br />
    default: the guest additions and repackage the box to continue.<br />
    default:<br />
    default: This is not an error message; everything may continue to work properly,<br />
    default: in which case you may ignore this message.<br />
==&gt; default: Configuring and enabling network interfaces...<br />
==&gt; default: Rsyncing folder: /Users/klenkes/Documents/demo/vagrant/ =&gt; /vagrant<br />
hermes:vagrant klenkes$<br />
[/code]</p>
<p>kann man die Box starten. Wie man aber sieht, können keine Verzeichnisse per Virtualbox gemountet werden, da die guest additions nicht installiert sind. Darum müssen wir uns kümmern:</p>
<h4>Wie kommen die Virtualbox Guest-Addons ins System?</h4>
<p>Hier gibt es ein Vagrant-Plugin namens "vagrant-vbguest". Und das installieren wir erstmal:</p>
<p>[code lang="bash" toolbar="true" title="Installieren von vagrant-vbguest"]<br />
hermes:vagrant klenkes$ vagrant plugin install vagrant-vbguest<br />
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...<br />
Installed the plugin 'vagrant-vbguest (0.13.0)'!<br />
hermes:vagrant klenkes$<br />
[/code]</p>
<p>Die Meldung "this can take a few minutes..." ist ernst gemeint. Es kann etwas dauern. Ist das Plugin installiert, muss es noch konfiguriert werden - natürlich im Vagrantfile:</p>
<p>[code lang="text" toolbox="true" title="Virtualbox Guest-Addons konfigurieren im Vagrantfile"]<br />
...<br />
  if Vagrant.has_plugin?(&quot;vagrant-proxyconf&quot;)<br />
    config.vbguest.auto_update = true<br />
    config.vbguest.no_remote = false<br />
  end<br />
...<br />
[/code]</p>
<p>Beim nächste start sollte es jetzt da sein. Das Plugin wird erstmal den GCC und alles was CentOS noch so braucht, um Kernelmodule zu kompilieren, installieren und dann auch gleich die Addons installieren. Sehr bequem.</p>
<h3>Pack den Proxy in den Tank</h3>
<p>Meistens befindet man sich ja als Consultant in einem abgeschlossenen Netz und braucht einen Proxy, um auf das Internet zugreifen zu könne. Hier hilft das Plugin "vagrant-proxyconf". Mittels</p>
<p>[code lang="bash" toolbar="true" title="Installieren von vagrant-proxyconf"]<br />
hermes:vagrant klenkes$ vagrant plugin install vagrant-proxyconf<br />
Installing the 'vagrant-proxyconf' plugin. This can take a few minutes...<br />
Installed the plugin 'vagrant-proxyconf (1.5.2)'!<br />
hermes:vagrant klenkes$<br />
[/code]</p>
<p>lässt sich das benötigte Modul installieren. Nun kann man den Proxy in das Vagrantfile eintragen und die Einstellungen aus dem Host-System in die Box übernehmen:</p>
<p>[code lang="text" toolbox="true" title="Proxy-Konfiguration im Vagrantfile"]<br />
...<br />
  if Vagrant.has_plugin?(&quot;vagrant-proxyconf&quot;)<br />
    puts &quot;Configuring proxy!&quot;<br />
    if ENV[&quot;http_proxy&quot;]<br />
      puts &quot;http_proxy: &quot; + ENV[&quot;http_proxy&quot;]<br />
      config.proxy.http = ENV[&quot;http_proxy&quot;]<br />
    end<br />
    if ENV[&quot;https_proxy&quot;]<br />
      puts &quot;https_proxy: &quot; + ENV[&quot;https_proxy&quot;]<br />
      config.proxy.https = ENV[&quot;https_proxy&quot;]<br />
    end<br />
    if ENV[&quot;no_proxy&quot;]<br />
      puts &quot;no_proxy: &quot; + ENV[&quot;no_proxy&quot;] + &quot;,127.0.0.1,localhost&quot;<br />
      config.proxy.no_proxy = ENV[&quot;no_proxy&quot;] + &quot;,127.0.0.1,localhost&quot;<br />
    end<br />
...<br />
[/code]</p>
<h3>Mount des Verzeichnisses /vagrant</h3>
<p>Laut Dokumentation wird dieses Verzeichnis bei virtualbox per Virtualbox gemounted. Leider wurde es bei mir immer nur per rsync synchronisiert. Aber ein Eintrag für den Mount mit dem richtigen Typ hat dies behoben:</p>
<p>[code lang="text" toolbox="true" title="Mountpoint korrigieren"]<br />
...<br />
   config.vm.synced_folder &quot;./&quot;, &quot;/vagrant&quot;, type: &quot;virtualbox&quot;<br />
...<br />
[/code]</p>
<p>Damit steht einem das Host-Verzeichnis auch in der Gast-Box zu Verfügung.</p>
<h3>Build-System</h3>
<p>Jetzt muss man noch das gewünschte Build-System per Provision installieren. Ich installiere git und java gleich mit. und habe es in einem Aufwasch hinter mir. In das Script kann man noch alle notwendigen Änderungen einfügen. Es handelt sich um ein Shell-Script, dass mit den Rechten des Users "vagrant" auf der Box ausgeführt wird. Root-Aktionen müssen also per "sudo" eingeleitet werden ...</p>
<p>Die Sourcen legen wir in unserem Host-Directory neben das Vagrantfile und nennen den Ordner src. Damit steht er sowohl auf dem Host wie auch in der Guest-Box zu Verfügung.</p>
<p>[code lang="text" toolbox="true" title="Build-System aufsetzen"]<br />
...<br />
  config.vm.provision &quot;shell&quot;, inline: &lt;&lt;-SHELL<br />
     sudo yum install -y git unzip zip vim java-1.8.0-openjdk-headless java-1.8.0-openjdk-devel-debug maven</p>
<p>     cd /vagrant</p>
<p>     mkdir -p /vagrant/src<br />
  SHELL<br />
...<br />
[/code]</p>
<p>Und voila, wir haben eine Vagrant-Box, die ein definiertes Build-System beinhaltet.</p>
<p>Und alles zusammen sieht das Vagrantfile nun so aus:</p>
<p>[code lang="text" toolbox="true" title="Vollständiges Vagrantfile für ein Build-System"]<br />
# -*- mode: ruby -*-<br />
# vi: set ft=ruby :</p>
<p># All Vagrant configuration is done below. The &quot;2&quot; in Vagrant.configure<br />
# configures the configuration version (we support older styles for<br />
# backwards compatibility). Please don't change it unless you know what<br />
# you're doing.<br />
Vagrant.configure(&quot;2&quot;) do |config|<br />
  config.vm.box = &quot;centos/7&quot;</p>
<p>  # accessing &quot;localhost:8080&quot; will access port 80 on the guest machine.<br />
  config.vm.network &quot;forwarded_port&quot;, guest: 80, host: 8080<br />
  config.vm.network &quot;private_network&quot;, ip: &quot;192.168.33.10&quot;</p>
<p>  # Share an additional folder to the guest VM. The first argument is<br />
  # the path on the host to the actual folder. The second argument is<br />
  # the path on the guest to mount the folder. And the optional third<br />
  # argument is a set of non-required options.<br />
  # config.vm.synced_folder &quot;../data&quot;, &quot;/vagrant_data&quot;<br />
  config.vm.synced_folder &quot;./&quot;, &quot;/vagrant&quot;, type: &quot;virtualbox&quot;</p>
<p>  # Provider-specific configuration so you can fine-tune various<br />
  # backing providers for Vagrant. These expose provider-specific options.<br />
  # Example for VirtualBox:<br />
  #<br />
  config.vm.provider &quot;virtualbox&quot; do |vb|<br />
     # Display the VirtualBox GUI when booting the machine<br />
     vb.gui = false</p>
<p>     # Customize the amount of memory on the VM:<br />
     vb.memory = &quot;1024&quot;<br />
  end</p>
<p>  if Vagrant.has_plugin?(&quot;vagrant-proxyconf&quot;)<br />
    puts &quot;Configuring proxy!&quot;<br />
    if ENV[&quot;http_proxy&quot;]<br />
      puts &quot;http_proxy: &quot; + ENV[&quot;http_proxy&quot;]<br />
      config.proxy.http = ENV[&quot;http_proxy&quot;]<br />
    end<br />
    if ENV[&quot;https_proxy&quot;]<br />
      puts &quot;https_proxy: &quot; + ENV[&quot;https_proxy&quot;]<br />
      config.proxy.https = ENV[&quot;https_proxy&quot;]<br />
    end<br />
    if ENV[&quot;no_proxy&quot;]<br />
      puts &quot;no_proxy: &quot; + ENV[&quot;no_proxy&quot;] + &quot;,127.0.0.1,localhost&quot;<br />
      config.proxy.no_proxy = ENV[&quot;no_proxy&quot;] + &quot;,127.0.0.1,localhost&quot;<br />
    end<br />
  end</p>
<p>  # Enable provisioning with a shell script. Additional provisioners such as<br />
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the<br />
  # documentation for more information about their specific syntax and use.<br />
  config.vm.provision &quot;shell&quot;, inline: &lt;&lt;-SHELL<br />
     sudo yum install -y git unzip zip vim java-1.8.0-openjdk-headless java-1.8.0-openjdk-devel-debug maven</p>
<p>     cd /vagrant</p>
<p>     mkdir -p /vagrant/src<br />
  SHELL<br />
end<br />
[/code]</p>
<p>Viel Spaß damit!</p>
