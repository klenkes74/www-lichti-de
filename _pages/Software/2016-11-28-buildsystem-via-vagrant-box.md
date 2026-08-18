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
In manchen Java-Projekten hat man das Problem, dass Entwickler unter Windows, Linux und MacOS arbeiten. Damit stellt schon das Build-Tooling ein Problem dar. Obwohl moderne Build-Systeme meistens in der JVM laufen und damit eigentlich betriebssystemunabhängig sein sollten, stößt man oft an das Problem, dass Pfadangaben leider nicht so unabhängig sind.

Um trotzdem lokal bauen zu können, wäre ein System schön, das wie ein zentraler Build-Server fungiert. Aber halt lokal. Und hier kommt Vagrant ins Spiel ...

In den meisten Fällen dürfte der Buildserver auf einem Linuxsystem laufen, die sind billig und schnell aufgesetzt. Meistens ist das die Qualifikation dafür, da das Management für solchen "Overhead" kein Geld ausgeben will. Daher betrachte ich hier nur diesen Fall[ref name="dislike-windows"]Außerdem mag ich Windows und MacOS als Server nicht.[/ref].

Was braucht man alles?

1. Eine Virtualisierungsumgebung (z.B. VMWare oder VirtualBox).
2. Vagrant zum Erzeugen der Virtual Machine.
3. Zeit, Lust und Durchhaltevermögen zum Erstellen des Vagrantfiles für die lokale Box.

Ich benutze hier VirtualBox auf einem MacBook Pro. Zusammen mit der VirtualBox ist das eine nette Kombination. Vagrant gibt es auf als Download auf [https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)[ref name="vagrant-version"]Die aktuelle Version 1.8.7 für den Mac sind ca. 70 MB, je nach Netzwerk ist es also sofort da oder dauert etwas.[/ref]. In Version 1.8.7 ist wohl  das gelieferte curl defekt und muss gelöscht werden:

```bash
hermes:vagrant klenkes$ rm /opt/vagrant/embedded/bin/curl
hermes:vagrant klenkes$
```
Damit  funktionieren die Box-Downloads[ref]siehe [https://github.com/twobitcircus/rpi-build-and-boot/issues/25](https://github.com/twobitcircus/rpi-build-and-boot/issues/25)[/ref]. Danach kann man per

```bash
hermes:vagrant klenkes$ vagrant box add centos/7
==> box: Loading metadata for box 'centos/7'
 box: URL: https://atlas.hashicorp.com/centos/7
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) libvirt
2) virtualbox
3) vmware_desktop
4) vmware_fusion

Enter your choice: 2
==> box: Adding box 'centos/7' (v1610.01) for provider: virtualbox
 box: Downloading: https://atlas.hashicorp.com/centos/boxes/7/versions/1610.01/providers/virtualbox.box
==> box: Successfully added box 'centos/7' (v1610.01) for 'virtualbox'!
hermes:vagrant klenkes$
```
eine Boxdefinition herunterladen. Ein Update könnte man mit

```bash
hermes:vagrant klenkes$ vagrant box update --box centos/7
Checking for updates to 'centos/7'
Latest installed version: 1610.01
Version constraints: > 1610.01
Provider: virtualbox
Box 'centos/7' (v1610.01) is running the latest version.
hermes:vagrant klenkes$
```
ausgeführt werden.

Und mit

```bash
hermes:vagrant klenkes$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
 default: Adapter 1: nat
 default: Adapter 2: hostonly
==> default: Forwarding ports...
 default: 80 (guest) => 8080 (host) (adapter 1)
 default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
 default: SSH address: 127.0.0.1:2222
 default: SSH username: vagrant
 default: SSH auth method: private key
 default: Warning: Remote connection disconnect. Retrying...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
 default: No guest additions were detected on the base box for this VM! Guest
 default: additions are required for forwarded ports, shared folders, host only
 default: networking, and more. If SSH fails on this machine, please install
 default: the guest additions and repackage the box to continue.
 default:
 default: This is not an error message; everything may continue to work properly,
 default: in which case you may ignore this message.
==> default: Configuring and enabling network interfaces...
==> default: Rsyncing folder: /Users/klenkes/Documents/demo/vagrant/ => /vagrant
hermes:vagrant klenkes$
```
kann man die Box starten. Wie man aber sieht, können keine Verzeichnisse per Virtualbox gemountet werden, da die guest additions nicht installiert sind. Darum müssen wir uns kümmern:

#### Wie kommen die Virtualbox Guest-Addons ins System?

Hier gibt es ein Vagrant-Plugin namens "vagrant-vbguest". Und das installieren wir erstmal:

```bash
hermes:vagrant klenkes$ vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Installed the plugin 'vagrant-vbguest (0.13.0)'!
hermes:vagrant klenkes$
```
Die Meldung "this can take a few minutes..." ist ernst gemeint. Es kann etwas dauern. Ist das Plugin installiert, muss es noch konfiguriert werden - natürlich im Vagrantfile:

```text
...
 if Vagrant.has_plugin?("vagrant-proxyconf")
 config.vbguest.auto_update = true
 config.vbguest.no_remote = false
 end
...
```
Beim nächste start sollte es jetzt da sein. Das Plugin wird erstmal den GCC und alles was CentOS noch so braucht, um Kernelmodule zu kompilieren, installieren und dann auch gleich die Addons installieren. Sehr bequem.

### Pack den Proxy in den Tank

Meistens befindet man sich ja als Consultant in einem abgeschlossenen Netz und braucht einen Proxy, um auf das Internet zugreifen zu könne. Hier hilft das Plugin "vagrant-proxyconf". Mittels

```bash
hermes:vagrant klenkes$ vagrant plugin install vagrant-proxyconf
Installing the 'vagrant-proxyconf' plugin. This can take a few minutes...
Installed the plugin 'vagrant-proxyconf (1.5.2)'!
hermes:vagrant klenkes$
```
lässt sich das benötigte Modul installieren. Nun kann man den Proxy in das Vagrantfile eintragen und die Einstellungen aus dem Host-System in die Box übernehmen:

```text
...
 if Vagrant.has_plugin?("vagrant-proxyconf")
 puts "Configuring proxy!"
 if ENV["http_proxy"]
 puts "http_proxy: " + ENV["http_proxy"]
 config.proxy.http = ENV["http_proxy"]
 end
 if ENV["https_proxy"]
 puts "https_proxy: " + ENV["https_proxy"]
 config.proxy.https = ENV["https_proxy"]
 end
 if ENV["no_proxy"]
 puts "no_proxy: " + ENV["no_proxy"] + ",127.0.0.1,localhost"
 config.proxy.no_proxy = ENV["no_proxy"] + ",127.0.0.1,localhost"
 end
...
```
### Mount des Verzeichnisses /vagrant

Laut Dokumentation wird dieses Verzeichnis bei virtualbox per Virtualbox gemounted. Leider wurde es bei mir immer nur per rsync synchronisiert. Aber ein Eintrag für den Mount mit dem richtigen Typ hat dies behoben:

```text
...
 config.vm.synced_folder "./", "/vagrant", type: "virtualbox"
...
```
Damit steht einem das Host-Verzeichnis auch in der Gast-Box zu Verfügung.

### Build-System

Jetzt muss man noch das gewünschte Build-System per Provision installieren. Ich installiere git und java gleich mit. und habe es in einem Aufwasch hinter mir. In das Script kann man noch alle notwendigen Änderungen einfügen. Es handelt sich um ein Shell-Script, dass mit den Rechten des Users "vagrant" auf der Box ausgeführt wird. Root-Aktionen müssen also per "sudo" eingeleitet werden ...

Die Sourcen legen wir in unserem Host-Directory neben das Vagrantfile und nennen den Ordner src. Damit steht er sowohl auf dem Host wie auch in der Guest-Box zu Verfügung.

```text
...
 config.vm.provision "shell", inline: <<-SHELL
 sudo yum install -y git unzip zip vim java-1.8.0-openjdk-headless java-1.8.0-openjdk-devel-debug maven

cd /vagrant

mkdir -p /vagrant/src
 SHELL
...
```
Und voila, wir haben eine Vagrant-Box, die ein definiertes Build-System beinhaltet.

Und alles zusammen sieht das Vagrantfile nun so aus:

```text
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
 config.vm.box = "centos/7"

# accessing "localhost:8080" will access port 80 on the guest machine.
 config.vm.network "forwarded_port", guest: 80, host: 8080
 config.vm.network "private_network", ip: "192.168.33.10"

# Share an additional folder to the guest VM. The first argument is
 # the path on the host to the actual folder. The second argument is
 # the path on the guest to mount the folder. And the optional third
 # argument is a set of non-required options.
 # config.vm.synced_folder "../data", "/vagrant_data"
 config.vm.synced_folder "./", "/vagrant", type: "virtualbox"

# Provider-specific configuration so you can fine-tune various
 # backing providers for Vagrant. These expose provider-specific options.
 # Example for VirtualBox:
 #
 config.vm.provider "virtualbox" do |vb|
 # Display the VirtualBox GUI when booting the machine
 vb.gui = false

# Customize the amount of memory on the VM:
 vb.memory = "1024"
 end

if Vagrant.has_plugin?("vagrant-proxyconf")
 puts "Configuring proxy!"
 if ENV["http_proxy"]
 puts "http_proxy: " + ENV["http_proxy"]
 config.proxy.http = ENV["http_proxy"]
 end
 if ENV["https_proxy"]
 puts "https_proxy: " + ENV["https_proxy"]
 config.proxy.https = ENV["https_proxy"]
 end
 if ENV["no_proxy"]
 puts "no_proxy: " + ENV["no_proxy"] + ",127.0.0.1,localhost"
 config.proxy.no_proxy = ENV["no_proxy"] + ",127.0.0.1,localhost"
 end
 end

# Enable provisioning with a shell script. Additional provisioners such as
 # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
 # documentation for more information about their specific syntax and use.
 config.vm.provision "shell", inline: <<-SHELL
 sudo yum install -y git unzip zip vim java-1.8.0-openjdk-headless java-1.8.0-openjdk-devel-debug maven

cd /vagrant

mkdir -p /vagrant/src
 SHELL
end
```
Viel Spaß damit!
