# CHAPTER 2 - Podman CLI

### Working with containers

- Running Containers
    - `podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash`
        - Command breakdown:
            - podman run ⇒ run the container.
            - `-ti` ⇒ tells podman to hook up to the terminal.
            - `--rm` ⇒ tells podman to delete the container as soon as teh container exits
            - `bash` ⇒ bash will override the default command the image runs with.
        - Once you run a different container on the same image ‘httpd’, podman will skip the image pulling step, because at the first time podman copies the blobs and stores them in the local container storage.
    - `podman run -d --name myapp -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24`
        - Command breakdown:
            - `--name` ⇒ Specify the container name.
            - `-d (--detach)` ⇒ tells podman to detach the container after running.
            - `-p (--publish)` ⇒ tells podman to publish or bind the container port `8080` to the host port `8080` when the container is running.
                - Note that if you specify one port, podman will consider this port a container port and randomly picks a host port on which the container port is bound. use `podman port` to discover which ports are bound to a container.
                    
                    ```bash
                    [root@kit ~]$ podman port d893198d7ab0
                    8080/tcp -> 0.0.0.0:8080 #port 8080/tcp inside the container is bound to all of the host networks (0.0.0.0) at port 8080
                    ```
                    
- Stopping Containers
    - `podman stop myapp`
    - When stopping a container, Podman examines the running container and sends a stop signal, usually **SIGTERM,** to the primary proc ‘PID1’ of the container.
    - By default, Podman waits 10 seconds to clean up and commit changes before sending **SIGKILL** signal to the process, forcing the container to stop.
    - `podman run --stop-signal` ⇒ to change the default stop signal.
    - `--timeout(-t)` ⇒ sets the timeout; -t 0 sends the SIGKILL without waiting for the container to stop.
    - `--latest(-l)` ⇒ stop the last created container.
    - `--all` ⇒ stop all running containers.
- Starting Containers
    - podman start myapp
    - `--all` ⇒ Starts all the stopped containers in container storage
    - `--attach` ⇒ this attaches your terminal to the output of the container.
    - `-i` ⇒ attaches to tty to container
- Listing Containers
    - `podman ps`
        - `--all` ⇒ list all containers running/stopped state.
        - `--quiet` ⇒ only print the container ID.
        - `--size` ⇒ Return the amount of disk space currently used other for each container other than the image they are based on.
- Inspecting Containers
    - `podman inspect myapp`  this will return a large JSON file, this information are handed down to the OCI runtime to launch the container.
    - You can inspect a specific information by running the following command
    - `podman inspect --format ‘{{  .Config.Cmd }}’ myapp` ⇒ Get the default cmd.
    - `podman inspect —format ‘{{ .NetworkSettings.IPAdress }}’ myapp` ⇒ get the ipaddress.
        - `-l` ⇒ inspect last created container.
        - `--format` ⇒ used to extract particular field out of the JSON.
- Removing Containers.
    - podman rm myapp
        - `--all` ⇒ remove all existing containers
        - `--force` ⇒ tells podman to stop all the running container when removing.
- exec-ing into Containers.
    - to execute a command inside a container use the following command:
        - `podman exec myapp bash`
    - Another great utility with exec,e.g. the following command will create an index.html file.
    
    ```bash
    $ podman exec -i myapp bash -c 'cat > /var/www/html/index.html' << EOF
    <html>
    	<h1>hello world</h1>
    </html>
    EOF
    ```
    
- Creating an image from a Container
    - `podma commit <cont_name> <img_name>`
        - `--pause` ⇒ pauses a running container during the commit. “serves as `podman pause` command”.
        - `--change` ⇒ Allwos you to commit instruction on using the image; e.g. CMD, ENV, EXPOSE, WORKDIR, ONBUILD, LABEL..
    - Stop/pause the container before commiting the container to an OCI image.
    - After stopping the container you can use podman commit to take your application container, myapp, and commit it, creating a new image name myimage.
        
        ```bash
        $ podman commit myapp myimage
        Getting image source signatures
        Copying blob 24839d45ca45 skipped: already exists  
        Copying blob 869dee90d0e8 skipped: already exists  
        Copying blob 5eb4ca2c6faa skipped: already exists  
        Copying blob e614afcf9060 skipped: already exists  
        Copying blob ac313715f43f skipped: already exists  
        Copying blob 39bdb39cf5b4 done  
        Copying config 2da4f3d011 done  
        Writing manifest to image destination
        Storing signatures
        2da4f3d011ec448777e8acb46629e936c087a569a418948ba75c99a26a1a8266
        
        $ podman images 
        REPOSITORY                            TAG              IMAGE ID      CREATED         SIZE
        localhost/myimage                     latest           2da4f3d011ec  56 seconds ago  173 MB
        ```
        
    - Using the podman commit command to create an image is not a common method, the entire process of building container images can be scripted and automated using podman build.
- More Commands
    
    ![Untitled](Untitled%205.png)
    
    ![Untitled](Untitled%206.png)
    

### Summary

| Command  | Description | Args |
| --- | --- | --- |
| `podman run` | run a container.  | `-d`  Run container in background and print container ID
• `it` - Attach TTY.
• `-name` - Assign a name to the container.
• `-p` - Publish a container's port to the host.
• `-user` - run the container as a specific user. |
| `podman create`  | create a container without starting it. | `-name` - Assign a name to the container |
| `podman start` | start one or more containers. | `-all` - Start all containers. |
| `podman stop` |  stop one or more containers. | `-all` - Stop all containers. |
| `podman restart`  | restart one or more containers | `-all` - Restart all containers. |
| `podman kill` |  send a signal to one or more containers. | `-all` - Send signal to all containers. |
| `podman rm` | remove one or more containers. | • `-all` - Remove all containers.
• `-f` - Force removal of running containers. |
| `podman ps` |  list running containers. | `a` - Show all containers.
`q`- Only print the Container ID |
| `podman exec` | run a command in a running contai. | `it` - Attach TTY |
| `podman commit` | create a new image from a container. | `-change` - Apply Dockerfile instruction during commi |
| `podman pull` | pull an image from a registry. |  |
| `podman push` | push an image to a registry. |  |
| `podman login` / `logout` | log in/out to/from a registry. |  |

### Working with images

- Image layers (blobs).
    - `podman image tree <image_name>` ⇒ Shows layers of an image.
    - `podman image diff <image_name1> <image_name2>` ⇒ Shows the difference between 2 images (C) - Changed, (D) - Deleted, (A) - Added.
- Listing Images
    - `podman images --all` ⇒ lists all the images.
        - When an image is replaced by a newer image with the same tag, the perivous images is tagged as <none><none>. Hence, they called **dangling images**.
- Inspecting Images
    - `podman image inspect` ⇒ Outputs a large JSON array that includes daya used for the OCI image format specification.
    - use the `--format` flagag to get a particular field out of the JSON.
        - `podman image inspect --format '{{ .Config.Cmd }}' myimage`
        - `podman image inspect --format '{{ .Config.StopSignal }}' myimage`
    
- Pushing Images
    - `podman push <image_name> <dest>` ⇒ used to push an image from local to remote registry.
        - You have to `podman login` before pusihing. You can use `--username` flag to specify the username.
        - by default, the creds are stored in /run/user/$UID/containers/auth.json.
        
        ```bash
        [chxmxii@kit ~]$ cat /tmp/podman-run-1002/containers/auth.json 
        {
        	"auths": {
        		"quay.io": {
        			"auth": "BASE64=="
        		}
        	}
        ```
        
    
    ![Untitled](Untitled%207.png)
    
    ![Untitled](Untitled%208.png)
    
- Tagging Images
    - `podman tag myimage quay.io/chxmxii/myimage`
    - Notice that `localhost/myimage` and  `quay.io/chxmxii/myimage` have the same container ID.
    - Now you can push the image using the `podman push quay.io/chxmxii/myimage` commad.
    - `podman tag myimage quay.io/chxmxii/myimage quay.io/chxmxii/myimage:1.0`
    - is important to note that the tag “latest” doesn’t refer to the most up-to-date image in the repo.
- Removing Images
    - `podman rmi localhost/myimage`
    - Note that podman will remove any tags before getting rid of the image.
    - You can remove the image by specifiying is ID.
    - `podman image prune -a`  ⇒ This will remove all the dangling images. add `-f` option to get rid of all the images, even those used by a container.
        - Dangling images are images that no longer have a tag associated with them or a container using them.
    - `podman system df` ⇒ This will show all of the storage in your home dir used by podman.
- Pulling Images
    - to pull an image from a registry use the following command
        - `podman pull quay.io/chxmxii/myimage`  \ some noteable options
            - `--arch` ⇒ tells podman to pull an image for a different arch e.g. arm64 image.
            - `--quiet /-q` ⇒ tells podman to not print out the progress information.
- Seaching for Images
    - `podman search registry.access.redhat.com/httpd`
    - Some noteable options
        - `--no-trunc` ⇒ tells podman to show full description.
        - `--foramt` ⇒ tells podman to display a specific field.
- Mounting Images
    - Podman provides the `podman image mount docker.io/library/httpd`command to mount an image’s rootfs in a read-only mode without creating a container from it. This will help you to examine the content of your image.
        
        <aside>
        ⚠️
        
        Note that you will have to execute the `podman unshare` commaned first.
        
        - The reason is you need to enter a user-namespace and sperate mount namespace.
        - the `podman unshare` command uses the same kernel features as (nsenter & unshare) commands. It simplifies the process of creating and configuring namespaces and inserting process into the namespace.
        </aside>
        
        ```bash
        $ podman unshare
        $ mnt=$(podman image mount docker.io/library/httpd)
        $ ls $mnt
        ```
        
    - Now you can start examining your image. use the command `podman image unmount docker.io/library/httpd` to unmount the image and exist the unshare session.
    
- Building Images
    - Below you find most of instructions to be found in any **Containerfile.**
        
        
        | **Instruction** | **Description** |
        | --- | --- |
        | FROM | Specifies the base image from which the new image will be built. |
        | MAINTAINER | Specifies the name and email address of the person or team responsible for the image. |
        | LABEL | Specifies metadata about the image. |
        | EXPOSE | Specifies the ports that the container will listen on. |
        | ADD | Copies a file or directory from the host machine or from a URL to the container. |
        | COPY | Copies a file or directory from the build context to the container. |
        | RUN | Runs a command in the container. *Podman actually runs a container on the image.* |
        | CMD | Specifies the command that will be run when the container starts. |
        | ENTRYPOINT | Specifies the command that will be run when the container starts, and cannot be overridden by the user. |
        | WORKDIR | Sets the working directory for subsequent instructions. |
        | USER | Sets the user and group that will be used to run subsequent instructions. |
        | VOLUME | Creates a volume that can be used to store data that persists between container restarts. |
        | ARG | Defines an argument that can be used to customize the build process. |
        | ONBUILD | Defines a command that will be run when the image is used as a base image for another image. |
        | HEALTHCHECK | Runs a command to check the health of the container. |
        | SHELL | Defines the shell that will be used to run commands in the container. |
        | STOPSIGNAL | Specifies the signal that will be used to stop the container. |
        | ENV | Sets an environment variable in the container. |
        
    
    <aside>
    🆚 CMD vs ENTRYPOINT
    ⇒ the `CMD` instruction used to specify the default command that will be run when the container starts (can be **overwritten** by the user), and the `ENTRYPOINT` instruction used to specify the command that will always be run when the container starts (can’t be **overwritten**).
    
    </aside>
    
    <aside>
    🆚 ADD vs COPY
    
    | **Feature** | **ADD** | **COPY** |
    | --- | --- | --- |
    | Copies files from | The host machine or a URL | The build context |
    | More complex | Yes | No |
    | More powerful | Yes | No |
    | Best practice | Use `COPY` unless you need the extra features of `ADD` |  |
    </aside>
    
    - Example of Containerfile.
    
    ```docker
    #pulls the image from chxmxii repo
    FROM quay.io/chxmxii/image 
    #copies start.sh from host to container
    ADD start.sh /usr/bin/start.sh
    #copies index.html from the build context to the container
    COPY index.html /var/www/html/index.html
    #runs the yum command to update
    RUN yum -y update &&\
    		yum clean all
    #creats a mount point.
    VOLUME /var/lib/mydata
    #default command to run
    CMD /usr/bin/start.sh
    #hard setting command
    ENTRYPOINT ["bin","sh","-C"]
    #var env
    ENV foo="bar"
    #announce the port that the container will be exposing. (doesn't map or open any port)
    EXPOSE 8080
    #metadata
    LABEL Description="Hello world"
    #sets the author field
    MAINTAINER chxmxii
    #default stop signal
    STOPSIGNAL SIGTERM
    #sets the user name or UID
    USER apache
    #adds a trigger to be executed at a later time, when the image is used as the base for another build.
    ONBUILD
    #sets the working dirctory for run,cmd,entrypoint and copy directives. the directory will be created if it doesn't exist.
    WORKDIR /var/www/html
    ```
    
    - Now, you can build the image using the `podman build -t quay.io/chxmxii/image_2 .` command. It commits the image and tags (-t) it with quay.io/chxmxii/image_2, it is now ready to be pushed to the container registry.

### Summary

![Untitled](Untitled%209.png)

![Untitled](Untitled%2010.png)