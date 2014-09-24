# Docker Registry On Core

This is an example of seting up a private [skydns][SkyDNS] and a private [docker registry][Docker-Registry] on a [CoreOS][using-coreos] with VirtualBox and Vagrant.

## Streamlined setup

1) Install dependencies

* [VirtualBox][virtualbox] 4.3.10 or greater.
* [Vagrant][vagrant] 1.6 or greater.

2) Clone this project and get system up

	git clone https://github.com/xuwang/docker-registry.git
	cd docker-registry
	vagrant up

3) Login to the coreos vm and start the private docker registry

	vagrant ssh
	cd share/units
	./all start			# start all related service units
	./all status     	# check units status
	ping registry.service    # see if the service is registed

4) Push and pull images from the private docker registry

	docker pull scratch
	myscratch='registry.service:5000/scratch'
	docker tag scratch:latest $myscratch
	docker push  $myscratch
	docker search $myscratch
	docker rmi $myscratch

5) Clean it up

	exit # the coreos vm
	vagrant destroy

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
[using-coreos]: http://coreos.com/docs/using-coreos/
[SkyDNS]: https://github.com/skynetservices/skydns
[Docker-Registry]: https://github.com/docker/docker-registry


