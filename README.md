# vx-docker-image for Bebanjo CI

There are an standard solution to use docker on OS X [boot2docker](https://github.com/boot2docker/boot2docker) but is
not working for something or others. So we're going to use a vagrant VM with all what we need to build the docker images

### Prerequisites (Mac OS X)

1. [VirtualBox](https://www.virtualbox.org/)
2. [Vagrant](http://vagrantup.com/)
3. [Homebrew](http://brew.sh/)

### Instructions

#### Clone Docker-Packer

```sh
git clone https://github.com/aisrael/docker-packer
```

#### Install packer

```sh
brew install packer
```

#### Build the box

```sh
packer build template.json
```

#### Modification of the Vagrantfile

 - Comment out the docker port forwarding on the Vagrantfile
 - Set as synced_folder the path to this repo (vx-docker-image)

#### Launch the box

```sh
vagrant up
```

#### From now on we'll work inside the vagrant box

#### Move inside the box

```sh
vagrant ssh
```

#### Install packer (http://www.packer.io/intro/getting-started/setup.html)

```sh
# This can be slightly different of what you have to do
vagrant$ wget download-path
vagrant$ apt-get install unzip
vagrant$ unzip download_packer
# I've download the installation to the synced folder inside the vagrant VM
vagrant$ echo "export PATH=$PATH:/vagrant_data/packer-installation" >> /home/vagrant/.bash_profile
```

#### Export docker host

```sh
vagrant$ export DOCKER_HOST=tcp://127.0.0.1:4243
```

#### Change to the vagrant data folder

```sh
vagrant$ cd /vagrant_data
```

### Existing images

### precise

Default image build from an ubuntu:12.04 image, basic setup to be provisioned with ansible

```sh
vagrant@host:/vagrant_data$ ./build.sh precise
```

### precise-full

Do the same that the default precise image plus all the setup needed on the original vexor [repository](https://github.com/vexor/vx-docker-image).
Ansible playbook https://github.com/pacoguzman/vx-docker-image/blob/bebanjo/playbooks/site.yml

```sh
vagrant@host:/vagrant_data$ ./build.sh precise-full
```

### precise-vexor

This is the custom bebanjo image the use the precise-full resulting image as a base to add the bebanjo needs.
Ansible playbook https://github.com/pacoguzman/vx-docker-image/blob/bebanjo/playbooks/bebanjo.yml

```sh
vagrant@host:/vagrant_data$ ./build.sh precise-vexor
```

See [playbooks/git_repositories/tasks/main.yml](https://github.com/pacoguzman/vx-docker-image/blob/bebanjo/playbooks/git_repositories/tasks/main.yml) points to a src file .id_github.txt on that ignore file you have to paste
the ssh key of one of the github administrators to access to the repositories. Remember that file has not be on the public repository

### TODO

1. Use a variable for the uid of the docker user so it can read/write into the volumes, should be the same like
  the one for the vxworker user on the worker machines

2. Fix jruby installation (missing JAVA on the path)

3. Bootstrap bebanjo repositories directly on the image with ansible, or figure out other solution, for the moment
you can do the following after you build the image:

```sh
# start the image
workermachine$ docker run -i -t image_identifier /bin/bash

docker-image$ su - vexor
docker-image$ eval "$(ssh-agent)"
docker-image$ ssh-add ~/.ssh/id_rsa
# item are the big repos you want to cache, don't change the path where clone because the worker will use that specific path
docker-image$ git clone git@github.com:{{item}}.git /home/vexor/code/{{item}}
docker-image$ exit

# get the container id of the previous image and commit the changes (don't need to push the changes)
docker commit 085f27c6b1ec image_identifier
```

# update only installed ruby versions

```sh
# Update ruby-build
cd /opt/ruby-build && git pull

# Install the 2.1.5 ruby version
CONFIGURE_OPTS="--disable-install-doc" /opt/ruby-build/bin/ruby-build 2.1.5 /opt/rbenv/versions/2.1.5 && env RBENV_ROOT=/opt/rbenv rbenv rehash
env RBENV_ROOT=/opt/rbenv RBENV_VERSION=2.1.5 rbenv exec gem install bundler && env RBENV_ROOT=/opt/rbenv rbenv rehash
```
