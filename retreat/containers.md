# Containers: An introduction

## Session Info

**Driver**: Manuel de la Peña, Ciro Costa

**Date and Time**: TBA

## Agenda
* Why containers are so sexy?
* Differences with virtual machines
* Anatomy of a Container
* Docker Networking
* Persistence
* Isolating applications
* Exposing containers applications

### Why containers are so sexy?
Containers are rocking the IT scene, but why? We'll see a brief introduction to the benefits of using containers.

### Differences with virtual machines
But why not using virtual machines? Let's see the diferences between hardware virtualization and operative system virtualization.

### Anatomy of a Container
A container representes a reduced operative system, but how is it possible? We'll see how Linux capabilities allows isolating process hierarchies, network stacks, resources access and so on.

### Docker Networking
How Docker containers communicate with each others? Let's see how the network is set between Docker containers.

### Persistence
Stateful applications in volatile environments sounds a complex thing. Let's understand how Docker manages persistent data using volumes.

### Communicating containers and applications
Once our application is isolated, we need to communicate with the outside world, let's say processes' ports and data.
 
## Resources
* http://man7.org/linux/man-pages/man7/cgroups.7.html
* https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/ch01.html
* http://man7.org/linux/man-pages/man7/namespaces.7.html
* https://docs.docker.com/engine/userguide/networking/
* https://docs.docker.com/engine/userguide/networking/configure-dns/
* https://docs.docker.com/engine/userguide/networking/work-with-networks/#linking-containers-in-user-defined-networks
* https://goto.docker.com/rs/929-FJL-178/images/wp-understanding-docker-data-storage-rev3.pdf
* https://docs.docker.com/engine/userguide/networking/default_network/binding/