---
layout: article
permalink: /provisioning/vagrant_ansible_docker.html
title: Develop and deploy your php app upon Ansible and Docker
description: Diving into really developer friendly way to create and provision your development environment with the help of Ansible and Docker on top of a Vagrant VM instance.   
tags: [vagrant, docker, ansible, provisioning]
---

1. [Let Vagrant start it](#let-vagrant-start-it)


### 1. Let Vagrant start it.

It's should be actually fairly easy to bring your vm to run, simply start with a Vagrantfile 
that was already presented in [puppet provisioning](vagrant_puppet_docker.html#vagrant-configuration) post and extend
it with a reference to provision your vm with ansible and provide there your entry ansible playbook.

{% highlight ruby %}
#Vagrantfile

ip = "10.20.1.10"
hostname = "phping.local";

Vagrant.configure("2") do |config|

  config.vm.define hostname do |pagetion|

    pagetion.vm.box = "ubuntu/trusty64"

    pagetion.vm.hostname = hostname

    pagetion.vm.network "private_network", :ip => ip

    pagetion.hostmanager.enabled     = true
    pagetion.hostmanager.manage_host = true
    
    # where our provisioning will be managed 
    pagetion.vm.provision :ansible do |ansible|

      ansible.playbook = "provisioning/main.yml"

    end

  end

end

{% endhighlight %}

... to be continued. 