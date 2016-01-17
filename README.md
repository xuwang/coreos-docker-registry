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
	
It may take a while to pull related docker images depending on the network bandwidth.    
Have a cup of tea, and check the status with:

        systemctl status registry*
Output:

    ● registry-proxy.service - Docker Registry Proxy
       Loaded: loaded (/run/fleet/units/registry-proxy.service; linked-runtime)
       Active: active (running) since Fri 2014-09-26 05:49:24 UTC; 11s ago
       ...

    ● registry.service - Docker Registry
       Loaded: loaded (/run/fleet/units/registry.service; linked-runtime)
       Active: active (running) since Fri 2014-09-26 05:49:19 UTC; 5s ago
       ...
Wait until dots turn green, try:

	ping -c 2 registry.docker.local    # see if the service is registed
Output:

	PING registry.docker.local (172.17.8.101) 56(84) bytes of data.
	64 bytes from 172.17.8.101: icmp_seq=1 ttl=64 time=0.094 ms
	64 bytes from 172.17.8.101: icmp_seq=2 ttl=64 time=0.032 ms

Check if the service is protected with curl:
      
```
core@core-01 ~ $ curl -D - https://registry.docker.local/v2/
HTTP/1.1 401 Unauthorized
Server: nginx/1.9.9
Date: Sun, 17 Jan 2016 20:40:10 GMT
Content-Type: text/html
Content-Length: 194
Connection: keep-alive
WWW-Authenticate: Basic realm="Restricted"
```

Curl with user credentials:
```
core@core-01 ~ $ curl -D -  --user test:test https://registry.docker.local/v2/
HTTP/1.1 200 OK
Server: nginx/1.9.9
Date: Sun, 17 Jan 2016 20:42:53 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 2
Connection: keep-alive
Docker-Distribution-Api-Version: registry/2.0
X-Content-Type-Options: nosniff
```

### Push and pull images from the private docker registry
Try pushing busybox to the private registry:

	docker pull busybox
	mybusybox='registry.docker.local/test/busybox'
	docker tag busybox:latest $mybusybox
	docker push  $mybusybox
Output:

    The push refers to a repository [registry.docker.local/test/busybox] (len: 1)
    Sending image list

Well, with docker 1.2.0, it fails pushing a image _without_ error warning. 

Try to pull the image, the docker tells you what's the problem:

    Pulling repository registry.docker.local/test/busybox
Output:

    2014/09/24 23:49:38 Authentication is required.
Do login and try again:

	docker login https://registry.docker.local
	Username: test
	Password: test
	Email:
Output:

	Login Succeeded
 
_Note:_ For docker login/logout, the full url, e.g. _https://registry.docker.local_ is required.  

Push again:

    docker push $mybusybox
    
Output:

    The push refers to a repository [registry.docker.local/test/busybox] (len: 1)
    Sending image list
    Pushing repository registry.docker.local/test/busybox (1 tags)
    ...
    Pushing tag for rev [511136ea3c5a] on {https://registry.docker.local/v1/repositories/test/busybox/tags/latest}
    
### Clean it up

	exit # the coreos vm
	vagrant destroy

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
[using-coreos]: http://coreos.com/docs/using-coreos/
[SkyDNS]: https://github.com/skynetservices/skydns
[Docker-Registry]: https://github.com/docker/docker-registry


