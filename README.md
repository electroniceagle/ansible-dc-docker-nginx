# Docker and Ansible on Mac OSX

This is a quick tutorial on how to use docker and ansible on Mac OSX and make a container visible to the world.

The challenge: https://github.com/AnsibleDC/AutoDockIT

## Install homebrew

Homebrew (http://brew.sh/) is a package management system for OSX.  It requires XCode.  Paste the following into a terminal window:

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

## Install docker and boot2docker using homebrew

1) Install the cask (http://caskroom.io/) extension for homebrew:

    brew cask install virtualbox

2) Install docker and boot2docker:

    brew install docker
    brew install boot2docker

3) Follow the instructions to start boot2docker at login:

    To have launchd start boot2docker at login:
        ln -sfv /usr/local/opt/boot2docker/*.plist ~/Library/LaunchAgents
    Then to load boot2docker now:
        launchctl load ~/Library/LaunchAgents/homebrew.mxcl.boot2docker.plist

4) Download the boot2docker VM and start the boot2docker daemon:

    boot2docker init
    boot2docker up

5) Follow the instructions to setup your docker environment (your setup will be different):

    To connect the Docker client to the Docker daemon, please set:
        export DOCKER_HOST=tcp://192.168.59.103:2376
        export DOCKER_CERT_PATH=/Users/bschott/.boot2docker/certs/boot2docker-vm
        export DOCKER_TLS_VERIFY=1

Or, you can add this to your .bash_profile:

    $ eval "$(boot2docker shellinit)"


## Install ansible and docker dependencies

1) I recommend installing virtualenvwrapper for managing virtual environments:
https://virtualenvwrapper.readthedocs.org/en/latest/

2) Install Ansible 1.9 and docker-py:

    pip install docker-py
    pip install -e git+ssh://git@github.com/ansible/ansible.git@v1.9.0-0.2.rc2#egg=ansible

Note: There is a TLS support issue for the docker module, which has been resolved since Ansible 1.9.RC2 (https://github.com/ansible/ansible-modules-core/issues/657).


## Edit the ansible_python_interpreter argument.

1) If you are using a virtualenv, you need to set the ansible_python_interpreter to point to your environment.  Edit the inventory/localhost file:

    $ more inventory/localhost
    localhost ansible_python_interpreter=~/.virtualenvs/ansible-ec2-docker-nginx/bin/python


## Deploy and test the docker container

1) Execute the deploy playbook:

    $ ./deploy.yaml

    PLAY [make a docker container with nginx on mac os x] *************************

    TASK: [grab web content from github] ******************************************
    ok: [localhost]

    TASK: [unarchive the zip content to local files directory] ********************
    changed: [localhost]

    TASK: [make a local docker container] *****************************************
    changed: [localhost -> 127.0.0.1]

    PLAY RECAP ********************************************************************
    localhost                  : ok=3    changed=2    unreachable=0    failed=0

2) Check that the container is running:

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                           NAMES
    1469578031eb        nginx:1             "nginx -g 'daemon of   3 hours ago         Up 3 hours          443/tcp, 0.0.0.0:8880->80/tcp   nginx

3) Verify that the web site is running:

    open "http://`boot2docker ip`:8880"

Note the backquotes. On OS X, docker isn't running on localhost, so you have to connect to the virtualbox VM running the docker engine.

![screen shot 2015-03-25 at 1 58 15 pm](https://cloud.githubusercontent.com/assets/219202/6831519/28da0b64-d2f7-11e4-9b96-23dac08b9088.png)

## Make the port available to the host

1) Expose the 8880 port (used by this playbook) from the VM to OS X host:

    VBoxManage controlvm "boot2docker-vm" natpf1 "tcp-port8880,tcp,,8880,,8880";

2) Test the localhost connection:

    open "http://localhost:8880"

![screen shot 2015-03-25 at 2 13 13 pm](https://cloud.githubusercontent.com/assets/219202/6832001/17f07f74-d2fa-11e4-8d3b-8efcef8d0326.png)


3) Test the world connection:

    open "http://`curl ifconfig.me`:8880"

![screen shot 2015-03-25 at 2 21 03 pm](https://cloud.githubusercontent.com/assets/219202/6832032/399b5144-d2fa-11e4-9031-b8c67fb1851a.png)

Yeah, OK, I'm behind another layer of NAT, but you get the idea.

## Good References

- http://brew.sh/
- http://caskroom.io/
- http://penandpants.com/2014/03/09/docker-via-homebrew/
- http://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide
- https://docs.docker.com/installation/mac/
