# Docker Registry With Basic Auth On CoreOS

An example of setting up a private [skydns][SkyDNS] and [docker registry][Docker-Registry] on [CoreOS][using-coreos] with VirtualBox and Vagrant.

This example starts 2 docker containers:

* Registry v2, registered with Skydns as _registry.service_
* Nginx as a basic auth Registry proxy, with Skydns name _registry.docker.local_

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

	ping -c 2 registry.docker.local    # see if the proxy is running
	ping -c 2 registry.service         # see if the proxy upstream registry service is registered
	
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

__Note:__ Because the regsitry is using a self-signed certificate, curl the registry from the host machine, you will need to use "-k" option:

```
$ curl -k -D - --user test:test https://172.17.8.101/v2/
```

### Push and pull images from the private docker registry
Try pushing busybox to the private registry, without authentication:

	docker pull busybox
	mybusybox='registry.docker.local/test/busybox'
	docker tag busybox:latest $mybusybox
	docker push  $mybusybox
Output:

    The push refers to a repository [registry.docker.local/test/busybox] (len: 1)
    b175bcb79023: Image push failed
    Head https://registry.docker.local/v2/test/busybox/blobs/sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4:     no basic auth credentials
    
Do login and try again:

    docker login -u test -p test -e test@example.com https://registry.docker.local
	
Output:

    WARNING: login credentials saved in /home/core/.docker/config.json
    Login Succeeded
 
Push again:

    core@core-01 ~ $ docker push $mybusybox
    The push refers to a repository [registry.docker.local/test/busybox] (len: 1)
    b175bcb79023: Pushed
    583635769552: Pushed
    latest: digest: sha256:caaa1b651f40a420b45c937a2033c28774dfba966caea8d2b77500615be97858 size: 2740
    
### Clean it up

	exit # the coreos vm
	vagrant destroy

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
[using-coreos]: http://coreos.com/docs/using-coreos/
[SkyDNS]: https://github.com/skynetservices/skydns
[Docker-Registry]: https://github.com/docker/docker-registry


