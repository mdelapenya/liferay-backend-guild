# Why containers are so sexy?

Containers are rocking the IT scene, but why? We'll see a brief introduction to the benefits of using containers.

    - Simplicity of deployment: It becomes much easier to deploy your whole application onto any hardware.
    - Forget about the O.S. on each deployment.
    - Less Devops code to maintain the infrastructure, as it is standardized.
    - Containers should optimally provide a single service, so teams reduce the code they work on.
    - Containers should be ephemeral: if a container dies, a new one should be able to be brought up in its place with no additional configuration.
    - Resiliency increased: an exactly the same container can be started in place of the fallen one.

## Differences with virtual machines

But why not using virtual machines? Let's see the diferences between hardware virtualization and operative system virtualization.

| Virtual Machines| Containers |
| ----------------------- |--------------------|
| Hardware virtualization | O.S. virtualization |
| Need an hypervisor | Need a container runtime (Docker Engine) |
| Install full blown O.S. | Share the underlying kernel (Linux or Windows) |
| Use resources before installing the application | Only exist when the application is run* |
| The application has to be installed in the O.S. | The application is declared inside the image |
| An application is one process of the many processes running inside a VM | An application is the `init` process (pid 1) of the container, and should be the only process running |

## Anatomy of a Container

A container representes a reduced operative system, but how is it possible? We'll see how Linux capabilities allows isolating process hierarchies, network stacks, resources access and so on.

### Cgroups

Cgroups allow processes to be organized into hierarchical groups whose usage of various types of resources can then be limited and monitored. Examples of such resources can be CPU time, system memory, network bandwidth, or combinations of these resources — among user-defined groups of tasks (processes) running on a system. It's possible to monitor the cgroups you configure, deny cgroups access to certain resources, and even reconfigure your cgroups dynamically on a running system.

A cgroup is a collection of processes that are bound to a set of limits or parameters defined via the cgroup filesystem.

A subsystem is a kernel component that modifies the behavior of the processes in a cgroup.  Various subsystems have been implemented, making it possible to do things such as limiting the amount of CPU time and memory available to a cgroup, accounting for the CPU time used by a cgroup, and freezing and resuming execution of the processes in a cgroup.  Subsystems are sometimes also known as resource controllers (or simply, controllers).

The cgroups for a controller are arranged in a hierarchy.  This hierarchy is defined by creating, removing, and renaming subdirectories within the cgroup filesystem.  At each level of the hierarchy, attributes (e.g., limits) can be defined.  The limits, control, and accounting provided by cgroups generally have effect throughout the subhierarchy underneath the cgroup where the attributes are defined.  Thus, for example, the limits placed on a cgroup at a higher level in the hierarchy cannot be exceeded by descendant cgroups.

Child tasks created by tasks under a cgroup hierachy inherit the exact same cgroups its parent task belongs to. But once forked, parent and child process are completely independent.

### Namespaces

A namespace wraps a global system resource in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource.  Changes to the global resource are visible to other processes that are members of the namespace, but are invisible to other processes.  One use of namespaces is to implement containers.

#### IPC (CLONE_NEWIPC): System V IPC, POSIX message queues

Objects created in an IPC namespace are visible to all other processes that are members of that namespace, but are not visible to processes in other IPC namespaces.

#### Network (CLONE_NEWNET): Network devices, stacks, ports, etc

Network namespaces provide isolation of the system resources associated with networking: network devices, IPv4 and IPv6 protocol stacks, IP routing tables, firewalls, the `/proc/net` directory, the `/sys/class/net` directory, port numbers (sockets), and so on.  A physical network device can live in exactly one network namespace.  A virtual network device ("veth") pair provides a pipe-like abstraction that can be used to create tunnels between network namespaces, and can be used to create a bridge to a physical network device in another namespace.

#### Mount (CLONE_NEWNS): Mount points

Mount namespaces provide isolation of the list of mount points seen by the processes in each namespace instance.  Thus, the processes in each of the mount namespace instances will see distinct single-directory hierarchies

#### PID (CLONE_NEWPID): Process IDs

PID namespaces isolate the process ID number space, meaning that processes in different PID namespaces can have the same PID.  PID namespaces allow containers to provide functionality such as suspending/resuming the set of processes in the container and migrating the container to a new host while the processes inside the container maintain the same PIDs.

PIDs in a new PID namespace start at 1, somewhat like a standalone system, and calls to fork(2), vfork(2), or clone(2) will produce processes with PIDs that are unique within the namespace.

#### User (CLONE_NEWUSER): User and group IDs

User namespaces isolate security-related identifiers and attributes, in particular, user IDs and group IDs (see credentials(7)), the root directory, keys (see keyctl(2)), and capabilities (see capabilities(7)).  A process's user and group IDs can be different inside and outside a user namespace.  In particular, a process can have a normal unprivileged user ID outside a user namespace while at the same time having a user ID of 0 inside the namespace; in other words, the process has full privileges for operations inside the user namespace, but is unprivileged for operations outside the namespace.

User namespaces can be nested; that is, each user namespace —except the initial ("root") namespace— has a parent user namespace, and can have zero or more child user namespaces.  The parent user namespace is the user namespace of the process that creates the user namespace.

Each process is a member of exactly one user namespace.

#### UTS (CLONE_NEWUTS): Hostname and NIS domain name

UTS namespaces provide isolation of two system identifiers: the hostname and the NIS domain name.

### Process hierarchies

In UNIX, all processes are descendents of the `init` process, whose PID is 1. The kernel starts `init` in the last step of the boot process. The `init` process, in turn, reads the system initscripts and executes more programs, eventually completing the boot process.

Every process on the system has exactly one parent. Likewise, every process has zero or more children. Processes that are all direct children of the same parent are called siblings. The relationship between processes is stored in the process descriptor. Each task_struct has a pointer to the parent's task_struct, named parent, and a list of children, named children.

### Docker Image format

#### Layers

Let's see an example: If a developer was building a Java-based web application, they would start with a base operating system. On top of that they would install Java, and finally their application code. This general process is the same whether it’s being done with Docker, on a physical box, or in a VM. However, in a case where Docker is not being used, the resulting files and subdirectories from each installation step would all be intermingled. All of the installed components are written to a single monolithic file system. By contrast, with Docker each step results in a new, isolated, read-only layer being added to the image. For instance, every Docker image starts with some base operating system layer. As components are added to this image, new layers are created. Each layer separates the newly added components from any components that were previously installed. The underlying layers remain untouched. If a newly added layer includes a file or directory that exists in a lower layer, the top most instance of that file or directory takes precedence. This is one of the fundamental ways Docker helps address scenarios like library mismatches.

When a container is instantiated, a read/write layer is added on top of the underlying read-only image layers. Any changes made to the container while it is running, for instance a log file being written, are reflected in the container layer; the underlying image layers are never affected. As soon as a change is initiated to an underlying layer, a new read-write layer is added. This is referred to as copy on write. When a new container is started, Docker does not clone the read-only base image, it simply creates a new container layer and links that to the existing base image.

Docker will not only share the base image between containers, but it will also share the same layers between different images.

Docker ships with a default Docker storage driver enabled by default. The specific driver is distribution dependent, for instance for Ubuntu it’s AUFS whereas for Red Hat it’s device mapper. In addition to the two drivers previously mentioned (AUFS and device mapper) Docker also supports the following Docker storage drivers: OverlayFS, Btrfs, VFS, ZFS.

Since Docker will only store a given image layer once, one best practice is to settle on a defined set of core images. IT operations staff should work to provide their development staff with a consistent set of base images.

## Docker Networking

How containers communicate with each others? Let's see how the network is set between Docker containers.

There are three default networks when running Docker containers: `bridge`, `none` and `host`.

### brigde

The bridge network represents the docker0 network present in all Docker installations. Unless you specify otherwise with the `docker run --network=<NETWORK>` option, the Docker daemon connects containers to this network by default.

The Docker Engine automatically creates a Subnet and Gateway to the network. The docker run command automatically adds new containers to this network.

Containers in this default network are able to communicate with each other using IP addresses.

Docker does not support automatic service discovery on the default bridge network. If you want to communicate with container names in this default bridge network, you must connect the containers via the legacy docker `run --link` option.

### none

The none network adds a container to a container-specific network stack. That container lacks a network interface.

### host

The host network adds a container on the hosts network stack. The network configuration inside the container is identical to the host.

### User-defined networks

You can create multiple networks. You can add containers to more than one network. Containers can only communicate within networks but not across networks. A container attached to two networks can communicate with member containers in either network. When a container is connected to multiple networks, its external connectivity is provided via the first non-internal network, in lexical order.

#### bridge

The containers you launch into this network must reside on the same Docker host. Each container in the network can immediately communicate with other containers in the network. Though, the network itself isolates the containers from external networks. Linking is not supported, but you can expose and publish container ports on containers in this network. This is useful if you want to make a portion of the bridge network available to an outside network.ç

#### docker_gwbridge

It's automatically created by Docker in two circumstances: when you initialize or join a swarm, Docker creates the docker_gwbridge network and uses it for communication among swarm nodes on different hosts. Or, when none of a container’s networks can provide external connectivity, Docker connect the container to the docker_gwbridge network in addition to the container’s other networks, so that the container can connect to external networks or other swarm nodes.

#### overlay

##### Docker Swarm mode

The swarm makes the overlay network available only to nodes in the swarm that require it for a service. When you create a service that uses the overlay network, the manager node automatically extends the overlay network to nodes that run service tasks. Overlay networks for a swarm are not available to containers started with docker run that don’t run as part of a swarm mode service.

##### external key-value store

Supported key-value stores include `Consul`, `Etcd`, and `ZooKeeper` (Distributed store). Before creating a network on this version of the Engine, you must install and configure your chosen key-value store service. The Docker hosts that you intend to network and the service must be able to communicate. Once connected, each container has access to all the containers in the network regardless of which Docker host the container was launched on.

#### embedded DNS server

It provides automatic service discovery for containers connected to user defined networks. Name resolution requests from the containers are handled first by the embedded DNS server. If the embedded DNS server is unable to resolve the request it will be forwarded to any external DNS servers configured for the container. To facilitate this when the container is created, only the embedded DNS server reachable at `127.0.0.11` will be listed in the container’s resolv.conf file

### Links

Containers can be discovered by its name automatically. But you can still create links but they behave differently when used in the default docker0 bridge network compared to user-defined networks. `--link <name or id>:alias` will link a container, even if it does not exist (if so, once the container is created, the link starts working).

Docker automatically creates environment variables in the target container based on the --link parameters. It will also expose all environment variables originating from Docker from the source container, including `ENV`, `-e`, `--env` and `--env-file`

Docker adds a host entry for the source container to the /etc/hosts file.

Creating Networks: When you create a network, Engine creates a non-overlapping subnetwork for the network by default. You can override this default and specify a subnetwork directly using the `--subnet` option. On a bridge network you can only specify a single subnet. An overlay network supports multiple subnets.

When you specify an IP address in this way while using a user-defined network, the configuration is preserved as part of the container’s configuration and will be applied when the container is reloaded. Assigned IP addresses are preserved when using non-user-defined networks, because there is no guarantee that a container’s subnet will not change when the Docker daemon restarts unless you use user-defined networks.

## Persistence

Stateful applications in volatile environments sounds a complex thing. Let's understand how Docker manages persistent data using volumes.

While a container is ephemeral, so is the data inside the container. If a container dies, anything in the read-write layer is lost. With Docker data persistence is achieved through volumes.

### Volumes

Images and containers are managed by the Docker storage driver. The storage driver stores all the layers under a designated subdirectory on the Docker host. For instance, with AUFS this directory is typically `/var/lib/docker`. Anything written to a container is stored inside of this directory structure, and is managed by the Docker storage driver.

A volume is simply a directory on the Docker host that exists outside of the directory structure managed by the storage driver. The directory is mapped from the Docker host to a directory inside the container. Anything written to or read from that directory bypasses the storage driver, and operates at host speeds. When a container is removed, the volume persists and the data is still available.

Multiple containers can mount to the same volume, but Docker does not do anything to control simultaneous access to the volume, it's up to the application to ensure data integrity.

Volumes can be backed up and restored using standard processes and tools.

Unless a volume is explicitly deleted when a container is removed, they will remain on the host.

## Communicating containers and applications

Once our application is isolated, we need to communicate with the outside world, let's say processes' ports and data.

By default Docker containers can make connections to the outside world, but the outside world cannot connect to containers. For that, the Docker server creates a `masquerade` rule that let containers connect to IP addresses in the outside world.

As we saw before, on Docker Networking, the network we connect a container is very important to determine which containers it can communicate with. We can isolate the container/application using the `none` network, or simply chosing a different network each time we create a container.

If you want containers to accept incoming connections, you will need to provide special options when invoking docker run:

    - `-P` or `--publish-all=true|false`: identifies every port with an EXPOSE line in the image’s Dockerfile
    - `--expose <port>`: maps it to a host port somewhere within an ephemeral port range (typically from 32768 to 61000).
    - `-p SPEC` or `--publish=SPEC`: It allows you to particularize which port on docker server - which can be any port at all, not just one within the ephemeral port range

If you want to be more restrictive and only allow container services to be contacted through a specific external interface on the host machine:

    - `-p IP:host_port:container_port` or `-p IP::port`: to specify the external interface for one particular binding.