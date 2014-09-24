# Preparing Your Mac for CoreOS-Vagrant

##Install [Homebrew][homebrew]

The missing package manager for OS X.

	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

##Install [VirtualBox][virtualbox] 4.3.10 or greater

With Homebrew, it’s trivial to install Virtualbox which is a prerequisite to running vagrant on OSX.

	brew update
	brew tap phinze/homebrew-cask
	brew install brew-cask
	brew cask install virtualbox

##Install [Vagrant][vagrant] 1.6 or greater

Don't know what Vagrant for? You must be kidding, get out of here!

##Install etcdctl and fleetctl
   
	brew install go etcdctl fleetctl
	etcdctl --version
	fleetctl version

Set fleet tunnel for vagrant coreos:

        export FLEETCTL_TUNNEL=localhost:$(vagrant ssh-config | grep Port | head -1 | awk '{print $2}')
	
You need to fresh fleetctl known_hosts after vagrant destroy and up:

	rm ~/.fleetctl/known_hosts


[homebrew]: http://brew.sh/
[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
