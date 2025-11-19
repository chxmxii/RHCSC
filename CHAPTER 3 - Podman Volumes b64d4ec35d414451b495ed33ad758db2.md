# CHAPTER 3 - Podman Volumes

<aside>
<img src="https://www.notion.so/icons/report_lightgray.svg" alt="https://www.notion.so/icons/report_lightgray.svg" width="40px" /> Bind mounts allows the same content to be accessible in two places, without any additional overhead. Is important to understand that bind doesn’t copy the data or create new data.

</aside>

- Volumes are useful for separating the data used by a container from the application inside an image.
- Volumes mount parts of the fs into a container’s env, which means security concerns like SELinux and user namespace need to be modified to allow access.
- Container Volumes
    - Podman allows you to mount host filesystem content into containers using the following command
        - `podman run -v HOST-DIR:CONTAINER-DIR` ⇒ the -v optioons tells podman to bind mount `HOST-DIR` in the host to `CONTAINER-DIR` in the container. Changes of the content on the host will be seen inside the container and vice-verca.
        - e.g.
            - `mkdir html && echo “Hello buddy” > /html/index.html`
            - `podman run -d -v ./html:/var/www/html:ro,z -p 8080:8080 httpd-24`
                - `ro` ⇒ mounts the volume in read-only mode. r/w by default.
                - `z` ⇒ tells podman to relabel the content to a shared label for use by SELinux.
                
                ⇒ If you curl [localhost:8080](http://localhost:8080) you will get **“Hello buddy”.**
                
- Named Volumes
    - use the command `podman volume create myvolume` to create a named volume.
    - you can inspect the volume nad look for its mount point using the `podman volume inspect myvolume` command.
    - `podman run -d -v myvolume:/var/www/html:ro,z -p 8080:8080 httpd` ⇒ Note that if the volume myvolume doesn’t exist, Podman will automatically create the volume.
    - use the `podman volume rm --force myvolume` command to remove the volume. the `--force` option was used to tell podman to remove the volume and all containers that rely on the volume.
    - use `podman volume list` to list all existing volumes.
    - use `podma volume export/import` command to export all the content of volume into an external TAR archive, or to recreate the volume on another machine using the TAR archive.
    
- Volume mount options
    - The `-U` options tells podman to recursively change ownership of the source voule to match the default UID the container executes with.
    `podman run --user mysql -v ./mariadb:/var/lib/mariadb:U docker.io/library/mariadb ls -ld /var/lib/mariadb.`
    
    | **Volume option** | **Description** |
    | --- | --- |
    | nodev | Prevent container processes from using character or block devices on the volume. |
    | noexec | Prevent container processes from direct execution of any binaries on the volume. |
    | nosuid | Prevent SUID applications from changing their privilege on the volume. |
    | O | Mount the directory from the host as a temporary storage using the overlay filesystem. Modifications to the mount point are destroyed when the container finishes
    executing. This option is useful for sharing the package cache from the host into
    the container to allow speeding up builds. |
    | rw|ro | Mount a volume in read-only (ro) or read-write (rw) mode. By default, read/write is
    implied |
    | U | Use the correct host UID and GID based on the UID and GID within the container.
    Use with caution because this will modify the host filesystem. |
    | z|Z | Relabel file objects on the shared volumes. Choose the z option to label volume
    content as shared among multiple containers. Choose the Z option to label content
    as unshared and private. |