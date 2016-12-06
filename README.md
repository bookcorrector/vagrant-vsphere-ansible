Before read this document you must install and configure your Fedora24 desktop with vagrant and ansible.

This document explain how to install and configure vagrant to use Vmware Vcenter. Firstly we must install and configure our virtual environment. I have 2 ESXI servers worked with clustered storage from FC storage.

###The configuration of vCenter as following:
[![N|Solid](file:///home/jshahverdiev/vsphere/images/vcenter-structure.png)](https://nodesource.com/products/nsolid)

Create folder for our Vagrantfile and go this folder:
```sh
[jshahverdiev@cons2 ~]$ mkdir vsphere ; cd vsphere/
```

Create temps folder for file syncronization and tasks folder for ansible playbooks:
```sh
[jshahverdiev@cons2 ~]$ mkdir tasks/; cd temps/
```

Create cos7-playbook.yml file with the following content(This file will include install_nginx.yml file from tasks folder to install/configure and start nginx):
```sh
[jshahverdiev@cons2 vsphere]$ cat cos7-playbook.yml 
---
- hosts: all
  become: true
  tasks:
    - include: 'tasks/install_nginx.yml''
```

Create tasks/install_nginx.yml file with the following content:
```sh
[jshahverdiev@cons2 vsphere]$ cat tasks/install_nginx.yml 
- name: NGINX | Installing NGINX repo rpm
  yum:
    name: http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
 
- name: NGINX | Installing NGINX
  yum:
    name: nginx
    state: latest
 
- name: NGINX | Starting NGINX
  service:
    name: nginx
    state: started 
```

Install needed plugins:
```sh
[jshahverdiev@cons2 vsphere]$ vagrant plugin install vagrant-vsphere  
[jshahverdiev@cons2 vsphere]$ vagrant plugin install vagrant-guests-photon
```

Create and add new box for vsphere:
```sh
[jshahverdiev@cons2 vsphere]$ curl -k https://raw.githubusercontent.com/nsidc/vagrant-vsphere/master/example_box/metadata.json -O
[jshahverdiev@cons2 vsphere]$ tar cvzf vsphere-dummy.box ./metadata.json 
./metadata.json
[jshahverdiev@cons2 vsphere]$ vagrant box add vsphere-dummy ./vsphere-dummy.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'vsphere-dummy' (v0) for provider: 
    box: Unpacking necessary files from: file:///home/jshahverdiev/vsphere/vsphere-dummy.box
==> box: Successfully added box 'vsphere-dummy' (v0) for 'vsphere'!
```

Look at box files:
```sh
[jshahverdiev@cons2 vsphere]$ vagrant box list 
ub14x64       (virtualbox, 0)
vsphere-dummy (vsphere, 0)
```

Create vagrantfile with the following contents:
```sh
[jshahverdiev@cons2 vsphere]$ cat Vagrantfile 
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'vsphere'
current_dir = File.dirname(File.expand_path(__FILE__))

Vagrant.configure("2") do |config|
  config.nfs.functional = false
  config.vm.define "cos7" do |cos7|
    cos7.vm.provider :vsphere do |vsphere|
      # The IP address or name of Vcenter server
      vsphere.host = '10.50.94.10'
      # The domain name of ESXI hosts
      vsphere.compute_resource_name = 'Cluster'
      # The name of Resource pool which we created before
      vsphere.resource_pool_name = 'dev'
      # The name of Template file which we created before
      vsphere.template_name = 'cos7box'
      # The name of new virtual machine
      vsphere.name = 'centos7.vagbox'
      # Username of vcenter 
      vsphere.user = 'administrator@vsphere.local'
      # Password for vcenter
      vsphere.password = 'A123456789a!'
      vsphere.insecure = true
    end
    cos7.vm.box = "vsphere-dummy"
    # The name of box file which we already created
    cos7.vm.hostname = "cos7"
    # Playbook file which will install and configure nginx server
    cos7.vm.provision :ansible do |ansible|
      ansible.playbook = "cos7-playbook.yml"
    end
    # Syncronize folder to the virtual machine
    cos7.vm.synced_folder "#{current_dir}/temps", "/home/vagrant/temps", owner: "vagrant", group: "vagrant"
  end
end  
```

Use the following command to start new virtual machine and install nginx to this virtual machine(If you want to debug use the vagrant up --debug command):
```sh
[jshahverdiev@cons2 vsphere]$ vagrant up
```

Try to login to the virtual machine:
```sh
[jshahverdiev@cons2 vsphere]$ vagrant ssh cos7
```
