# Docker Registry On CoreOS

An example of setting up a private [skydns][SkyDNS] and [docker registry][Docker-Registry] on [CoreOS][using-coreos] with VirtualBox and Vagrant.

### Install dependencies

* [VirtualBox][virtualbox] 4.3.10 or greater.
* [Vagrant][vagrant] 1.6 or greater.

### Clone this project and get system up

	git clone https://github.com/xuwang/coreos-docker-registry.git
	cd docker-registry
	vagrant up

### Login to the coreos vm and start the private docker registry

	vagrant ssh
	cd share/units
	./all start			# start all related service units
	./all status     	# check units status
	ping registry.service    # see if the service is registed

### Push and pull images from the private docker registry

	docker pull scratch
	myscratch='registry.service:5000/scratch'
	docker tag scratch:latest $myscratch
	docker push  $myscratch
	docker search $myscratch
	docker rmi $myscratch

### Clean it up

	exit # the coreos vm
	vagrant destroy

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
[using-coreos]: http://coreos.com/docs/using-coreos/
[SkyDNS]: https://github.com/skynetservices/skydns
[Docker-Registry]: https://github.com/docker/docker-registry


