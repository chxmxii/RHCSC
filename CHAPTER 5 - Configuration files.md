# CHAPTER 5 - Configuration files

> Podman allows its users to have ultimate control over the defaults configurations, so the container engine configuration files provide a mechanism for customizing the way podman run.

### Configuration files for storage

- The storage library provides methods for storing filesystem layers, container images, and containers.
- The configuration of this file is done using the storage.conf file.
- run the command `podman info --format "{{ .Store.ConfigFile }}"`
- The most common reasons for changes is relocating the container’s storage.

### Configuration files for location

- storage.conf file calls the storage location the **graphRoot.**
- the location of the graphRoot can be overridden in /etc/containers/storage.conf for rootful containers.
- to change the location of the graphRoot edit the following line.
    - `graphroot = "/var/lib/containers/storage”`
- for rootless users edit the following line.
    - `rootless_storage_path = "$HOME/.local/share/containers/storage”`

<aside>
❗ Note that you should inform SELinux (if it is enabled) that you changed the default location of storage so it will relabel the new location as if it was in the old location.
+ `semanage fcontext -a -e /var/lib/containers/storage /var/mystorage` 
+ `restorecon -R -v /var/mystorage`
ROOTLESS MODE:
+ `sudo semanage fcontext -a -e $HOME/.local/share/containers/storage /var/tmp/$UID/var/mystorage`
+ `sudo restorecon -R -v /var/tmp/$UID/var/mystorage`

</aside>

### Configuration files for drivers

![storage drivers](assets/csd.png)

![storage drivers](assets/csd.png)

### Configuration files for registries

- registries.conf configuration file is a system-wide configuration file for container image registries. Podman uses `$HOME/.config/containers/registries.conf` if exists, otherwise, it uses `/etc/containers/registries.conf`

![registries](assets/regs.png)

- **Block pulling from container registries:**
    - you can configure the registries.conf to block pulling from any registry you want.
        
        ```toml
        [[registry]]
        Location = "docker.io"
        blocked = true	
        ```
        
    - [[registry]] is a table entry that can specify how to handle individual container registries.
        
        ![reg table](assets/regcli.png)
        

### Configuration files for engines

![conf file](assets/cconf.png)

```bash
$ mkdir -p ~/.config/containers/containers.conf.d/ && cd $_ && cat << _E > env.conf 
> [containers]
> env=[ "foo=bar" ]
> _E
$ podman run --rm ubi8 printenv
#you will notice the foo=bar within the env vars.
-------------------------------
$ podman run quay.io/podman/stable cat /etc/containers/containers.conf
[containers]
netns="host"
userns="host"
ipcns="host"
utsns="host"
cgroupns="host"
cgroups="disabled"
log_driver = "k8s-file"
[engine]
cgroup_manager = "cgroupfs"
events_logger="file"
runtime="crun"
```

### **System configuration files**

- /etc/subuid and /etc/subgid specifies the UID ranges for your containers. Podman reads these files and then launches /usr/bin/newuidmap and /usr/bin/newgidmap
    
    ![uuids](assets/uuids.png)
    
- use the command `podman system migrate` to stop the `podman pause` process and re-creates the new user-namespace and take then effects.

### Summary

- Podman has multiple configuration files based on the libraries it uses.
- Configuration files are share between rootful and rootless env.
- The **storage.conf** file is used to configure containers/storage, including the storage driver as well as the location where cotainer and their images are to be stored.
- The **registries.conf** and **policy.json** files are used to configure the container/image library. Primarily affecting access to container registries, shortnames, and mirror sights.
- The **containers.conf** file is used to configure all of the other defaults used within Podman.
- System configuration files lie **/etc/subuid** and **/etc/subgid** are used to configure the user-namespace required for running rootless containers.