# podman-installation-ansible-role
Ansible role - Install Podman with non-root(or un privileged run)

![image](https://user-images.githubusercontent.com/97512751/209323712-8c06fa62-480a-4930-bb7a-92869b271808.png)

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

## Sample nginx run
In the remote machine: just after the completion of the above process login,

```
podman run -itd -p 8080:80 --name webserver nginx
Resolving "nginx" using unqualified-search registries (/etc/containers/registries.conf)
Trying to pull docker.io/library/nginx:latest...
Getting image source signatures
Copying blob f12443e5c9f7 done
Copying blob ec0f5d052824 done
Copying blob 025c56f98b67 done
Copying blob defc9ba04d7c done
Copying blob 885556963dad done
Copying blob cc9fb8360807 done
Copying config 3964ce7b84 done
Writing manifest to image destination
Storing signatures
926f8a64858e1b757139720f766ee13519da41b799b98d538f07eed159857f6f
```

### Enable service at boot as a normal user.

Make sure the container is running,
```
$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS             PORTS                 NAMES
926f8a64858e  docker.io/library/nginx:latest  nginx -g daemon o...  21 seconds ago  Up 21 seconds ago  0.0.0.0:8080->80/tcp  webserver
```

Generate service file
```
$ podman generate systemd --new webserver -f
/home/ck/container-926f8a64858e1b757139720f766ee13519da41b799b98d538f07eed159857f6f.service
```

Copy the generated service move to .config/systemd/user/ .
```
Rename the name as per your naming convention and move the service file to the directory.
$mv -v /home/ck/container-926f8a64858e1b757139720f766ee13519da41b799b98d538f07eed159857f6f.service   .config/systemd/user/webserver.service
Renamed /home/ck/container-926f8a64858e1b757139720f766ee13519da41b799b98d538f07eed159857f6f.service -> .config/systemd/user/webserver.service
```
## Generate Systemd based service
```
$source .bashrc
$systemctl --user daemon-reload
$systemctl enable --user webserver.service
Created symlink /home/ck/.config/systemd/user/default.target.wants/webserver.service → /home/ck/.config/systemd/user/webserver.service.
```
Make a reboot and you can see the service is actively running.

## Result
Podman installed and ready for a non-root user to execute podamn, podman compose, and docker commands.

Note: 
docker is an alias to podman here.
Firewalld has to be opened in each case.
if a service is depend on anther service, we can use the following command to make sure the dependancy is available.
```sh
$podman generate systemd --requires=<name_of_dependent_service_to_start_before> --new <new service> -f
```
