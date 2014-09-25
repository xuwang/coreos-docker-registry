# Docker Registry With Basic Auth On CoreOS

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
	ping registry.docker.local    # see if the service is registed

### Push and pull images from the private docker registry

	docker pull scratch
	myscratch='registry.docker.local/test/scratch'
	docker tag scratch:latest $myscratch
	docker push  $myscratch
    
Well, with docker 1.2.0, it will fail and put out some nonsense:

    The push refers to a repository [registry.docker.local/test/scratch] (len: 1)
    Sending image list

Try pull the image, the docker tells you what's the problem:
 
    Pulling repository registry.docker.local/test/scratch
    2014/09/24 23:49:38 Authentication is required.
    
Do login and try again:

	docker login https://registry.docker.local
    Username: test
    Password: test
    Email:
    Login Succeeded
    
	docker push  $myscratch
    The push refers to a repository [registry.docker.local/test/scratch] (len: 1)
    Sending image list
    Pushing repository registry.docker.local/test/scratch (1 tags)
    Image 511136ea3c5a already pushed, skipping
    Pushing tag for rev [511136ea3c5a] on {https://registry.docker.local/v1/repositories/test/scratch/tags/latest}
    
_Note:_ docker (1.2.0) search is not working at all with basic auth, push is partially working :-()

Use the Docker Registry API call for search:

        curl --user test:test -s -XGET "https://registry.docker.local/v1/search?q=test
### Clean it up

	exit # the coreos vm
	vagrant destroy

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
[using-coreos]: http://coreos.com/docs/using-coreos/
[SkyDNS]: https://github.com/skynetservices/skydns
[Docker-Registry]: https://github.com/docker/docker-registry


