# CHAPTER 1 - Introducing Podman

- **Podman VS Docker**
    - **Integration with systemd** - Podman supports running systemd inside of the container while Docker does not.
    - **User namespaces:** Podman supports running containers in a separate user namespace.
    - **Daemonless:** Podman command runs like a traditional CLI tool, while docker requires multiple root-running daemons.
    - **Supports K8S-like pods:** Podman supports running multiple containers within the same pod.
    - **Customizable defaults**: Podman supports fully customizing all of its defaults including security, namespaces, volumes, and more.
- **Containers**
    - **What are containers?**
        - Basically, Containers are a group of processes running on a Linux system that is isolated from each other. It does make sure that one group of processes does not interfere with any other group on the system.
        - Unlike virtual machines, containers are not meant to host an operating system. yet, they are meant to run a specific task or porcess (.e.g; host an instance of web server, database).
        - A container only lives as long as the process inside it is alive. if the process is stopped or crashes, the container exits.
    - **Why use containers?**
        - Containers allow applications to be installed with their own versions of shared libraries that do not conflict with applications that require different versions of the same libraries.
        - Containers allow applications to live in a virtualized environment, giving the impression that they own the entire system.
    - **How Containers are isolated?**
        - **cgroups - Control groups:** a Linux kernel feature that allows processes to be organized into hierarchical groups whose usage of various types of resources can then be limited and monitored. e.g:
            - The amount of memory.
            - The amount of CPU.
            - The amount of network resources.
            
            ⇒ cgroups prevent containers from dominating certain system resources in such a way that another container can’t make progress on the system
            
        - **Security constraints:** to block rogue containers from committing malicious acts against the system or to block privilege escalation, Containers are isolated from each other using many security tools available in the kernel such as:
            - Dropped Linux capabilities → limit the power of root.
            - SELinux → controls access to the filesystem.
            - SECCOMP → limits the system calls available in the kernel.
        - **namespaces**: creates virtualized env, where one container sees one set of resources, while another container sees a different set of resources, giving them the feel of a virtual machine. e.g:
            - Network namespace → Eliminates access to the host network but gives access to the virtual network devices.
            - Mount namespace → Eliminates the view of all the filesystems, except the container’s filesystem.
            - PID namespace →  Eliminates the view of other processes on the system, except the processes within the container itself.
    - **Container images**
        - Container images solve the dependency management problem by bundling all the software requirements to run your application together into a unit. You ship all the executables, configuration files, and libraries together.
        - The software is isolated from the host via container technology. Usually, the only part of the host system that your application interacts with is the host kernel.
        - The concept of container images led to microservices.
        - In microservices, Containers communicate with each other via a virtual private network.
        - **Container image format:**
            - A dir tree containing all the software dependencies to run your application.
            - A JSON file that describes the contents of the rootfs.
            - A JSON file called a **manifest** list that links multiple images together to support different architectures.
            - `podman pull docker://registry.access.redhat.com/ubi8 && podman inspect docker://registry.access.redhat.com/ubi8` note that `podman inspect`works only on local images (already pulled).
            
            ```bash
            "Version": "",
                      "Author": "", #author
                      "Architecture": "amd64", #architecture for this image
                      "Os": "linux", # os the contianer is using
                      "Size": 214846503,
                      "VirtualSize": 214846503,
            					"Config": {
                           "Env": [ #env vars setted within the container
                                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                                "container=oci"
                           ],
                           "Cmd": [ #command to be executed once the container starts.
                                "/bin/bash"
                           ],
                      "RootFS": {
                           "Type": "layers",
                           "Layers": [
                                "sha256:b51194abfc91510e7cc523fd1735ea595e9ab7b8fbb8697e7e2ac6bb04b0f5e3"
                           ]
                      },
                      "Labels": { #lables are helpful to describe the ocntencts of the image
                           "architecture": "x86_64",
                           "build-date": "2023-06-23T02:02:46",
                           ...
            ```
            
            - Another great tool to inspect container images is skopeo, which provides a lower level out-put examining the structures of a container image manifest specificqtion. 
            `skopeo inspect --raw docker://registry.access.redhat.com/ubi8`
                
                ```bash
                {
                    "manifests": [
                        {
                            "digest": "sha256:45dfedd04e6778a86d54ca062237809fc033fbe5a6c81098f8bc9b6199d2f318",
                						#Digest of the exact image pulled when the architecture and OS match.
                            "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
                            "platform": {
                                "architecture": "amd64",#archi of this image difest
                                "os": "linux" #os of the image
                            },
                            "size": 429
                        },
                        {
                            "digest": "sha256:ba803ecfde6ad36bb739ab2858c0eea2e4c0257a42905ffa25018368079ec2f9",
                						#digest of the a different image for a different archi
                            "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
                						#mediatype describes the type of the image, OCI, docker etc..
                            "platform": {
                                "architecture": "arm64",
                                "os": "linux"
                            },...
                ```
                
- **Rootless Container**
    - The ability to run containers in the rootless is probably the most significant feature of podman. In many situations, you do not want to grant full root access to your users, yet users and developers need still need to run containers and build container images.
    - Requiring root access prevents lots of security-conscious companies from using Docker. Podman, on the other hand, can run containers with no additional security features in Linux other than a standard login account.
    - `$ docker run -it --name badman --privileged -v /:/host ubi8 chroot /host`
        - the `--privileged` flag turns off all container security.
        - we mounted the entire host os / on the /host dir within the container and then we chroot to /host, BOOM!! you are in a root shell at/of the os with full root privilege.
    - NOTE: Docker now has the ability to run rootless similarly to Podman.
        - Docker and podman architecture
            
            ![docker arch](assets/dockerarch.png)
            
    
    ![podman arch](assets/parch.png)
    
- **REST API**
    - Podman can be run as a socket-activated REST API service which will allows remote clients to manage and launch Podman containers.
    - Podman supports the Docker API as well.
    - The Podman REST API also allows remote Podman clients on MAC, Windows, and Linux systems to interact with Podman containers on a Linux machine.
- **Pods**
    - Podman works with either a single container at a time, like docker, or it can manage groups of containers togethere in a pod.
    - A pod is a group of one ore more containers, with shared sotrage/network resources.
    
    ![Figure P.1 : Two pods running on a host. Each pod runs two different containers along with the infra container.](assets/layers.png)
    
    Figure P.1 : Two pods running on a host. Each pod runs two different containers along with the infra container.
    
    - `podman generate kube` ⇒ allows you to generate kubernetes YAML files from running containers and pods.
    - `podman play kube` ⇒ allows you to play k8s yaml files and generate pods and containers on your host.
    
    ⇒ is recommended to use Podman for running pods and containers on single host. k8s can be used to run pods and containers on multiple machines and all through your infrastrucutre.
    
- **Customized Registries**
    - Podman supports the concept of pulling images using short names, such as **ubi8** without specifying the registry they reside `registry.access.redhat.com/library/ubi8:latest`
        
        ![registry](assets/reg.png)
        
    - While in docker, if you attempt to pull ubi8/httpd-24 it will fail because the container is not on [docker.io](http://docker.io), and thats because Docker is hardcoded to always pull from [https://docker.io](https://docker.io) when using a short name.
    - Unlike docker, Podman allow you to specify multiple registries, like what you do with package managers. If you attempt to pull ubi8/httpd-24, podman will present you a list of registries to choose from.
    
- **Transports**
    - Podman supports many different container image sources and targets called transports.
    - Podmcan can pull images from container registries and from local containers storage.
        
        ![pull](assets/pull.png)
        
- **Customizability**
    - Unlike other container engines, Podman has a very customizable configuration.
    - Podman configuration files are stored in :
        - `/usr/share/containers/containers.conf` ⇒ where a distribution can define the changes the distribution likes to use.
        - `/etc/containers/containers.conf` ⇒ Set up system overrides.
        - `$HOME/.config/containers/containers.conf` ⇒ can be specified only in rootless mode.
    
    ⇒ The configuration files allow you to configure Podman to run the way you want by default.
    
- **User-namespaces**
    - user-namespace allows for multiple UIDs to be assigned to a user, providing isolation between users on a system.
    - Podman makes it easy and simple to launch multiple containers, each with a unique user namespace. The kernel then isolates the processes  from host users as well as each other based on UID separation.
    - Docker, on the other hand, supports only running containers in a single, sepearate, usernamespace. meaning root in one container is the same as root in another container. It does not support running each container in a different user namespace, which means container attack each other from a username prespective.

## Summary

- Podman is container enginer, suitable for almost all of your single-node container projects. It is useful for developing, building, and running containerized applications.
- Podman is as simple to use as Docker.
- Podman supports a REST API, which allows remote tools and languages, including dokcer-compose, to work with Podman containers
- Podman noteable features:
    - user-namespace support, multiple transports, customizable registries, integration with systemd, fork/exec model, rootless mode
- Podman is more secure way to run containers.