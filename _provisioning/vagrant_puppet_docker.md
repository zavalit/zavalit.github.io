---
layout: article
permalink: /provisioning/vagrant_puppet_docker.html
title: Develop your php app upon Vagrant, Puppet and Docker
description: A result oriented "howto" to develop and run a php application with Vagrant, Puppet and Docker.
tags: [vagrant, docker, puppet, provisioning]
---

The point of this post is to show how we can develop localy a php app, 
that suites an both local and production topologies. 

#### Vagrant setup

First of all prepare an actual vagrant package from [Vagrant download page](http://www.vagrantup.com/downloads).
At this point we dicide to build our system on top of ubuntu 14.04, for that purpose we load an appropriate
vagrant box


{% highlight bash %}

    $ vagrant box add ubuntu/trusty64

{% endhighlight %}



That should load an ubuntu image, that vagrant will provide for virtualisation. Why do we ever need all that, in a case if we have ubuntu 14.04 
as a host? We should have a max possible control of our environment, in a case we mess it along with out host system, you can be never sure of all possible side effects your host system have apart.

{% highlight ruby %}
# empty ubuntu image with hostname phping 
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "phping"  

end 

{% endhighlight %}


... to be continued
