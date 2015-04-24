---
layout: article
permalink: /provisioning/vagrant_puppet_docker.html
title: Develop your php app upon Vagrant, Puppet and Docker
description: A result oriented "howto" to develop and run a php application with Vagrant, Puppet and Docker.
tags: [vagrant, docker, puppet, provisioning]
---

1. [Vagrant configuration](#vagrant-configuration)
2. [Puppet configuration](#puppet-configuration)
3. [Run automated tests with serverspec](#run-automated-tests-with-serverspec)

The point of this post is to show how we can develop localy a php app, 
that suites an both local and production topologies. 

### 1. Vagrant configuration 

First of all prepare an actual vagrant package from [Vagrant download page](http://www.vagrantup.com/downloads).
At this point we dicide to build our system on top of ubuntu 14.04, for that purpose we load an appropriate
vagrant box


{% highlight bash %}

    $ vagrant box add ubuntu/trusty64

{% endhighlight %}


That should load an ubuntu image, that vagrant will provide for virtualisation. Why do we ever need all that, in a case if we have ubuntu 14.04 
as a host? We should have a max possible control of our environment, in a case we mess it along with out host system, you can be never sure of all possible side effects your host system have apart.

{% highlight ruby %}
# empty ubuntu image with hostname phping.local 
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "phping.local"  

end 

{% endhighlight %}


It can be optional, but it makes definitely sense to control an ip that both our vagrant vm a host machine are
aware about. For that we install an vagrant plugin:

{% highlight bash %}

    $ vagrant plugin install vagrant-hostmanager

{% endhighlight %}

and extend our Vagrantfile with an ip, that we want our vagrant instance should have. In parallel
the Vagrantfile could be refactored a bit, in order to make our vagrant instance more specific.

{% highlight ruby %}

# prepare variables
ip = "10.20.1.10"
hostname = "phping.local";

# run config
Vagrant.configure("2") do |config|

  # here we gave a vagrant vm instance a name "phping.local" that is referenced by variable hostname
  config.vm.define hostname do |phping|
  
    phping.vm.box = "ubuntu/trusty64"
    phping.vm.hostname = hostname
  
    phping.vm.network "private_network", :ip => ip
 
    phping.hostmanager.enabled    = true
    phping.hostmanager.manage_host = true

  end

end 

{% endhighlight %}


and afterwards apply the hostname and ip with:

{% highlight bash %}
# extend host's /etc/hosts with an vagrant hostname
$ vagrant hostmanager
    
# apply an ip, with which we can speak to our vagrant ubunutu virtual machine
$ vagrant reload

{% endhighlight %}

On that point, the only thing that is still to do is to let vagrant to know about our 
puppet. For that we extend our Vagrantfile with provision statement:

{% highlight ruby %}

# ...box, hostname and networking settings 
  
  phping.vm.provision :puppet do |puppet|

    puppet.module_path = ["modules", "vendor/modules"] # where do we store our own and external puppet modules
    puppet.options     = '--debug --verbose --summarize --reports store' # some debug

  end

# ... Vagrantfile end

{% endhighlight %}


and not to forget to create that module_path folders in the same folder. 


{% highlight bash %}

$ mkdir -p vendor/modules modules 

{% endhighlight %}

### 2. Puppet configuration
 
Now If try to run "phping.local" vm, it will fail, cause it needs some input to our puppet 
configuration.
Before we begin to write our own specific manifests we should take care about dependencies, 
that are necessary to let puppet run docker. In order to download them in a proper way we will
need puppet on our host machine. There are a bunch of ways how to do it, i propose to do it 
as a gem. We define a following Gemfile:


{% highlight ruby %}

source "https://rubygems.org"

gem "librarian-puppet"
gem "puppet"

{% endhighlight %}


and run bundler in the same folder

{% highlight bash %}

$ bundle install

{% endhighlight %}

Now, the only thing, before we can start to write our own model, is to define our docker dependency and download it as a model.
We create the following a Puppetfile in a current vagrant setup folder:

{% highlight ruby %}

forge "https://forgeapi.puppetlabs.com"

mod 'garethr/docker', :git => 'https://github.com/garethr/garethr-docker.git'

{% endhighlight %}

and execute an installation routine

{% highlight bash %}

$ librarian-puppet install --path /vendor/modules

{% endhighlight %}

From that point we are ready to write our docker service module. At first comes the suitable 
folder structure:
{% highlight bash %}

$ cd modules && mkdir -p application/manifests application/files

{% endhighlight %}

First of all we create an app that we want to run. I use here my experimental php server, cause it
simple enough to run and has no dependencies besides php itself. For that we create a folder "corouser" in 
 "modules/application/files" and then get it composer package:

{% highlight bash %}

$  cd modules/application/files/corouser && composer require --dev zavalit/corouser:dev-master

{% endhighlight %}


After composer get that package to our file system, the only thing is still to do is define our
application class to run that thing in a docker container. In modules/application/manifests comes for now an initial docker service manifest, we name it init.pp

{% highlight ruby %}

class application {

   include docker

   $application_dir = '/opt/corouser'

   file { $application_dir:
     ensure => directory,
     source => 'puppet:///modules/application/corouser/vendor',
     force  => true,
     recurse => true,
   }

   ::docker::image { 
    'zavalit/corouser':
     require    => File[$application_dir],
   }

   ::docker::run { 
    'corouser':
    image => 'zavalit/corouser',
    volumes => "/opt/corouser:/var/www",
    ports => ['8081:8081'],
    require => Docker::Image['zavalit/corouser'],
   }

}

{% endhighlight %}
  

And we are done. We simply reload our vagrant:

{% highlight bash %}

$  vagrant reload --provision

{% endhighlight %}

and when all goes through, we can call in a browser http://phping.local:8081 and see smth. like this:
{% highlight bash %}
Recieved following request:

GET / HTTP/1.1
Host: phping.local:8081
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:37.0) Gecko/20100101 Firefox/37.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: de,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: keep-alive

{% endhighlight %}



### 3. Run automated tests with serverspec

We are done, but! as an awesome developer, that we all are, since we try that stack out,
 we can not just test it manually. It's not the a way a proper software can be developed and supported.
  For that we extend our Gemfile with an additional gem serverspec:
 
{% highlight ruby %}

source "https://rubygems.org"

gem "librarian-puppet"
gem "puppet"
gem "serverspec"

{% endhighlight %}

 
and execute serverspec-init in a project root. The output tree should look smth. like this:

{% highlight bash %}

$ serverspec-init
# ... dialog how to configure your initial spec setup 

# and that is how it could be structured in the end
spec/
|____phping.local
| |____sample_spec.rb
|____spec_helper.rb

{% endhighlight %}


We replace a template sample_spec.rb with our own spec file base_spec.rb and write a spec to test whether our php server 
corouser is running

{% highlight ruby %}

require 'spec_helper'

[22, 8081].each do |value| 

  describe port(value) do
    it {should be_listening}
  end

end

describe host('phping.local') do
  it {should be_reachable}
  it {should be_resolvable}
end
{% endhighlight %}


Now it is a perfect time to let serverspec run all defined tests:

{% highlight bash %}

$ rake spec

Port "22"
  should be listening

Port "8081"
  should be listening

Host "phping.local"
  should be reachable
  should be resolvable

Finished in 1.77 seconds (files took 7.95 seconds to load)
4 examples, 0 failures


{% endhighlight %}

And **it {should make_me_happy}** to see that tests are passed. Now we can leave that setup in peace and take a bear. 

##### *if you experience some any problems to get it run you can take a look at [github repo](https://github.com/zavalit/vagrant_puppet_docker_stack) for it*
