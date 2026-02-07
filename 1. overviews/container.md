### Before Container (VM)

Topology
![](/assets/overviews/vm.png)

Issue:
- Compatibility / dependency
- Long setup time
- Different env



### Container solv the issue
Topology
![](/assets/overviews/container.png)

Benefits:
- Containerize app
- Run each service with it own dependencies in separate containers

---

### Container
is a completely isolated env

(virtual) own process / serice
(virtual) own network interface
(virtual) own mount

![](/assets/overviews/container-env.png)


### Sharing kernel
"Docker contianer is underlying kernel" = docker on base OS vm is ubuntu, it can run container base on linux distro eg. ubuntu, fedora, centos

![](/assets/overviews/sharekernel.png)



### VM vs Container

Docker container: 
- Underly hardware, OS
- docker container run with lib and dependencies alone.


VM:
- Underly hardware, OS and hypervisor
- hypervisor will create sub vm
- in sub vm will contain with os, lib, deps and application

![](/assets/overviews/container-vs-vm.png)

VM's utilize(+) > Docker container
VM's size(-) > Docker container
VM's boot up time(-) > Docker container