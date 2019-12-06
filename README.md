# docker-for-windows-virtualbox

Docker setup using virtualbox with SMB connection for better disk-io performance. Compared to native vagrant which hosts are volume from `Host > Guest`, this box aims to setup the box rom `Guest > Host`. With this swap Docker can use its native disk for better IO performance. Overall IDE's handle disk io and refreshes in memory pretty well.

## Prerequisites

* Virtualbox (www.virtualbox.org)
* Vagrant (www.vagrantup.com)

## Setup

Before starting the box, please review the configuration options so that the environment is set up correct from the start. Redoing it will probably be a waste of time :-)

```sh
vagrant up
```

## Usage

```sh
vagrant ssh
```

After all is up you can use `docker` native in your development box.

### Windows users 

For `Windows` users it is essential to run an `ssh agent`. To achieve this, put below snippet in you `GitBash` `.bashrc` file.

```shell script
# Edit your ~/.bashrc with below
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
```

### Volume mounting

To access code from your native host, a SMB share is available at `smb://<ip>/vagrantbox`. It mounts all working directories to `/home/vagrant/repositories`

### Configuration

#### SSH & GIT

To make life a little more easy, the SSH key and GIT config of the user that started the Vagrantbox are automaticly copied inside the box. Therefor you should be able to use your normal working flow like you are used to.

#### Box config

There is a `.env` file available that holds all settings. Feel free to adjust according your personal needs