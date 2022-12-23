# podman-installation-ansible-role
Ansible role - Install Podman with non-root(or un privileged run)

![image](https://user-images.githubusercontent.com/97512751/209323712-8c06fa62-480a-4930-bb7a-92869b271808.png)

![image](https://user-images.githubusercontent.com/97512751/209323378-4fceeec4-9b9a-4cae-9006-b95527ceff57.png)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Description
By using this ansible role you can install the podman, podman-docker and podman-compose. It will also configure a normal user to manage SELinux related persistance with temporary volumes, all other related configurations.
Podman service will be installed and ready for a non root user to run podman commands.

### Advantages of Podman over Docker
Docker uses a client-server architecture. Daemon runs behind the scenes where docker-cli provides instructions to docker engine.
Podman uses a single process architecture, due to this pods, images are smaller in size, it can avoid security issues due to multi-process architecture such as sharing PID namespaces with other containers, privilege escalation(docker uses root privileges behind the scenes) attacks and limited user provisioning with related to well-known ports or even ports in general.

A rootless container with inside and outside normal users makes the system one of the best to opt from. Podman provides such a great feature. By this the defence will be quite high against attackers.
### Similarity with Docker
Commands from docker is as par with commands which is using in docker. This makes the Podman an excellent choice of docker replacement.

## Pre requesites
* Rhel 8.x /CentOS 8.x /Rocky 8.x /Alma 8.x version with active SELinux and Firewalld running(this machine is the target machine which will be hosting the podamn).
* A normal user with “sudo” permission. (Can remove the user from the ‘sudoers’ after deployment).
* Need to open certain ports as per requirements of container exposed ports.
* Access to internet seemless. (Can remove after the installation).
* Firewall permission to be enabled.
> Note: 'ssh user@<hostname/host_IP>' add the future podman machine fingerprint to your local system

## Prepare the Ansible host machine.
Install the epel-release repo then install ansible
```sh
$dnf install -y epel-release
$dnf install -y ansible
```

Clone the repo to the local machine(hosts ansible host machine rhel7/8/9)
```sh
$git clone https://github.com/sudheernambiar/podman-installation-ansible-role.git
$cd ansbile-podman-installation-nonroot/
```

Now copy all contents inside the podman to /etc/ansible/roles/
```sh
cp -rvf podman /etc/ansible/roles/
```

Edit the ansible script podman_install.yaml with the sudo user you are using in this case, replace the user part > remote_user: ck to remote_user: <normal_username>.
```
-
  name: "Podman installation"
  hosts: all
  remote_user: ck
  roles:
    - podman

```
Run the ansible command as follows,
Change the IP of the machine with the remote host machine's IP/hostname which is your future podman host.
```sh
$ansible-playbook -i "192.168.29.211", podman_install.yaml -k -K
```

## Result
Podman installed and ready for a non-root user to execute podamn, podman compose, and docker commands.
Note: docker is an alias to podman here.
For systemd refer: 
