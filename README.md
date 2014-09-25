# Docker Registry On CoreOS

An example of setting up a private [skydns][SkyDNS] and [docker registry][Docker-Registry] on [CoreOS][using-coreos] with VirtualBox and Vagrant.

### Install dependencies

* [VirtualBox][virtualbox] 4.3.10 or greater.
* [Vagrant][vagrant] 1.6 or greater.

### Clone this project and get system up and login

	git clone https://github.com/xuwang/coreos-docker-registry.git
	cd docker-registry
	vagrant up
    vagrant ssh
	ping -c 2 registry.service    # see if the service is registed
    PING registry.service.docker.local (172.17.8.101) 56(84) bytes of data.
    64 bytes from 172.17.8.101: icmp_seq=1 ttl=64 time=0.094 ms
    64 bytes from 172.17.8.101: icmp_seq=2 ttl=64 time=0.032 ms
    ...

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


