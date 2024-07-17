# Container-Management-System

we developed a more complete container management tool like Docker. The tool will provide the following features:

Ability to create full fledged debian container images
Instantiating new containers from an image
Enabling following network capabilities for the containers
Network path between container and host
Network path between container and public
TCP Port forwarding between host and container
Network path between different containers on the host
There are two script files for this task config.sh and conductor.sh

config.sh is used to set configuration parameters for our tool
conductor.sh script is the actual tool that will perform all operaions


Functions provided by the tool are documented as follows:

build
Usage: ./conductor.sh build <image-name>
Operation: This function will download and create a debian system – the root directory structure of a debian system and all its dub-directories in a local directory, using the debootstrap command. The command will store the container image within the configured images directory as image-name

images
Usage: ./conductor.sh images
Operation: This function will list all container images available in the configured images directory.

rmi
Usage: ./conductor.sh rmi <image-name>
Operation: This function will delete the given image from the configured images directory.

run
Usage: ./conductor.sh <image-name> <container-name> -- [command <args>]
Operation: This function will start a new container with name as container-name from the specified image. command <args> is the program and it's arguments (if any) that will be executed within the container as the first process. If no command is given, it will execute /bin/bash by default. The container will have isolated UTS, PID, NET, MOUNT and IPC namespaces. It will also mount the following filesystems within the container:

procfs for tools dependent on it like ps, top etc. to work correctly
sysfs to be able to setup ip forwarding using iproute2 tool
/dev (in host) bind mount to container-rootfs/dev to enable the container to see network devices
The first process will have pid = 1 in the container. It will also set correct file permission on the container-rootfs/ so that tools like apt work correctly. A directory named container-name within the configured CONTAINERDIR directory will be created. Furthermore container-name will have a subdirectory named rootfs which will be mounted as root directory for the container.

Note: If you run a container and exit from it, you cannot run the same container again. First you will need to delete the container using stop command, then run it.

ps
Usage: ./conductor.sh ps
Operation: This function will show all running containers by querying entries within the configured CONTAINERDIR directory.

stop
Usage: ./conductor.sh stop <container-name>
Operation: This function will stop a running container with given name. Stopping a container involves:

Killing the unshare process that started the container
Killing all processes running within the container
Unmounting any remaining mount point within the contaier rootfs.
Deleting container-name with configured CONTAINERDIR directory.
Note: Stopping a container will delete all state of the container

exec
Usage: ./conductor.sh exec <container-name> -- [command <args>]
Operation: This function executes the given program along with it arguments within the specified running container. If no command is provided it will execute /bin/bash by default. The executed program will be in the same UTS, PID, NET, MOUNT and IPC namespace as the specified container. Furthermore it will see the root directory as conatiner-name/rootfs. It will also see the same procfs, sysfs and /dev filesystem as configured within the container and tools like ps, top etc. should work correctly.

addnetwork
Usage: ./conductor.sh addnetwork <container-name> [options]
Operation: This function will add a network interface to a container and setup its configurations so that the container can communicate using the network.

By default if no options is given it will setup the container network as shown in the diagram below:

+--------------------------------+                  +-----------------+
| Host                           |                  | container 'eg'  |
|                                |   From 'host'    |                 |
+-----------+       +------------+ TX->        RX-> +------------+    |
+ default   |       | eg-outside +------------------+  eg-inside |    |
+ interface |       +------------+ <-RX        <-TX +------------+    |
+---------- +                    |    From 'eg'     |                 |
|                                |                  |                 |
+--------------------------------+                  +-----------------+
It should be noted that only communication between eg-outside and eg-inside is possible. That implies that application running inside container eg will not be able to access the Internet (end points beyonde-outside.)

If the option -i or --internet is specified the script should allow for the applications running inside the container to access the Internet. The schematic diagram for the same is shown below:

+--------------------------------+                  +-----------------+
| Host                           |                  | container 'eg'  |
|                                |   From 'host'    |                 |
+-----------+       +------------+ TX->        RX-> +------------+    |
+ default   |--NAT--| eg-outside +------------------+  eg-inside |    |
+ interface |       +------------+ <-RX        <-TX +------------+    |
+---------- +                    |    From 'eg'     |                 |
|                                |                  |                 |
+--------------------------------+                  +-----------------+
Although the -i options allows Internet usage, exposing services deployed inside containers is still not possible (Basically, if you deploy a server or anything inside the container it will not be accessible outside the host).

You can use the --expose or -e option to make a port available to services outside of the container. This creates a rule in the host, mapping a container port to a port on the host to the outside world. Here are some examples:

./sudo conductor.sh addnetwork -e 8080-80 : Map port 80 on the host to TCP port 8080 in the container.

peer
Usage: ./conductor.sh peer <container1-name> <container2-name>
Operation: By default our conductor isolates the container so that no inter-container communication is possible. This function allows two container to communicate with each other.



