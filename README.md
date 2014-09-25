# Docker Registry With Basic Auth On CoreOS

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

_Note:_ docker (1.2.0) search is not working at all with basic auth, push is not helping when your are not logged in.  
Also, use the full the url `https://registry.docker.local` for docker login/logout.

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


