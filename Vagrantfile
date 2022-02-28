# @Author: Haibin Lee <haibin>
# @Date:   2020-11-11T16:56:37+08:00
# @Email:  haibin.li@newtouch.com
# @Filename: Vagrantfile
# @Last modified by:   haibin
# @Last modified time: 2020-12-04T13:28:18+08:00
# @Copyright: Copyright 2020 the original author or authors.



# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'fileutils'

Vagrant.require_version ">= 2.0.0"

# Uniq disk UUID for libvirt
DISK_UUID = Time.now.utc.to_i

SUPPORTED_OS = {
  "ubuntu1604"          => {box: "bento/ubuntu-16.04", user: "vagrant"},
  "official-ubuntu1604" => {box: "ubuntu/xenial64", user: "vagrant"},
  "official-ubuntu2004" => {box: "ubuntu/focal64", user: "vagrant"},
  "ubuntu1804"          => {box: "generic/ubuntu1804", user: "vagrant"},
  "centos"              => {box: "centos/7",           user: "vagrant"},
  "centos8"              => {box: "centos/8",           user: "vagrant"},
  "centos-bento"        => {box: "bento/centos-7.5",   user: "vagrant"},
  "fedora"              => {box: "fedora/28-cloud-base",                user: "vagrant"},
  "opensuse"            => {box: "opensuse/openSUSE-15.0-x86_64",       user: "vagrant"},
  "opensuse-tumbleweed" => {box: "opensuse/openSUSE-Tumbleweed-x86_64", user: "vagrant"},
}

# Defaults for config options defined in CONFIG
$num_instances = 10
$instance_name_prefix = "webase"
$vm_gui = false
$vm_memory = 4096
$vm_cpus = 2
$shared_folders = {}
$forwarded_ports = {}
$subnet = "10.17.8"
$os = "official-ubuntu2004"
$override_disk_size = false
$extra_disk_size = "20GB"

host_vars = {}

$box = SUPPORTED_OS[$os][:box]
# if $inventory is not set, try to use example
$inventory = "inventories/sample" if ! $inventory
$inventory = File.absolute_path($inventory, File.dirname(__FILE__))

if Vagrant.has_plugin?("vagrant-proxyconf")
    $no_proxy = ENV['NO_PROXY'] || ENV['no_proxy'] || "127.0.0.1,localhost"
    (1..$num_instances).each do |i|
        $no_proxy += ",#{$subnet}.#{i+100}"
    end
end

Vagrant.configure("2") do |config|

  config.vm.box = $box
  if SUPPORTED_OS[$os].has_key? :box_url
    config.vm.box_url = SUPPORTED_OS[$os][:box_url]
  end
  config.ssh.username = SUPPORTED_OS[$os][:user]

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  # always use Vagrants insecure key
  config.ssh.insert_key = false

  if ($override_disk_size)
    unless Vagrant.has_plugin?("vagrant-disksize")
      system "vagrant plugin install vagrant-disksize"
    end
    config.disksize.size = $extra_disk_size
  end

  # Disable default synced folder
  config.vm.synced_folder ".", "/vagrant", disabled: true

  (1..$num_instances).each do |i|
    config.vm.define vm_name = "%s-%01d" % [$instance_name_prefix, i] do |node|
      node.vm.network "forwarded_port", guest: 20200, host: 20200 + i

      node.vm.hostname = vm_name

      if Vagrant.has_plugin?("vagrant-proxyconf")
        node.proxy.http     = ENV['HTTP_PROXY'] || ENV['http_proxy'] || ""
        node.proxy.https    = ENV['HTTPS_PROXY'] || ENV['https_proxy'] ||  ""
        node.proxy.no_proxy = $no_proxy
      end

      node.vm.provider :virtualbox do |vb|
        vb.memory = $vm_memory
        vb.cpus = $vm_cpus
        vb.gui = $vm_gui
        vb.linked_clone = true
        vb.customize ["modifyvm", :id, "--vram", "8"] # ubuntu defaults to 256 MB which is a waste of precious RAM
        # Get disk path
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        line = `VBoxManage list systemproperties | grep "Default machine folder"`
        vb_machine_folder = line.split(':')[1].strip()
      end

      ip = "#{$subnet}.#{i+100}"
      node.vm.network :private_network, ip: ip

      if $i == 1
        node.vm.network :public_network
      end

      # Disable swap for each vm
      node.vm.provision "shell", inline: "swapoff -a"

      host_vars[vm_name] = {
        "ip": ip,
        "ansible_ssh_user": SUPPORTED_OS[$os][:user]
      }

    end
  end
end
