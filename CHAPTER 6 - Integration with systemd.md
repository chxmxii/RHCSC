# CHAPTER 6 - Integration with systemd

### Integration with systemd

![systemd](assets/systemd.png)

- You can run systemd within a container, this will allow you to run many services within a container.
    
    ![systemd-wcontainer](assets/sdwc.png)
    
- You can use **ubi8-init** image or any other image that runs the CMD `[/sbin/init, /usr/sbin/init,/usr/local/sbin/init,/*/systemd..]` as these commands will trigger podman to run in systemd mode.
- We can use the following containerfile to run httpd as a service within podman
    
    ```docker
    FROM ubi8-init
    RUN dnf -y install httpd; dnf -y clean all
    RUN systemctl enable httpd.service
    ```
    
    ```bash
    $ echo "Hello systemd" > html/index.html
    $ podman run -d --rm -p 8080:80 -v ./html:/var/www/html:Z my-systemd
    $ curl localhost:8080
    	Hello systemd
    ```
    

### Journald for logging and events

- **Log driver:**
    - run the command to see the default log driver on your system
        - `podman info --format '{{ .Host.LogDriver }}’`
    - To change the default log driver, create a file under `~/.config/containers/containers.conf.d/log_driver.conf`
        
        ```bash
        [containers]
        log_driver="journald"
        ```
        
        ![journald](assets/journald.png)
        
- **Events**
    - check the event records of the last container you ran:
        - `podman events --filter event=start --since 1h`
    - check the default events logger
        - `podman info --format '{{ .Host.EventLogger }}’`
    - to override the default event logger, create a file under `~/.config/containers/containers.conf.d/events_logger.conf`
        
        ![eventLogger](assets/elogger.png)
        

### Starting containers at boot

> Since podman does not run as a daemon, meaning you can’t rely on a daemon to automatically start your containers at boot time. Instead you will need to run containerized services via systemd.

- **Restart containers**
    - The `podman run` command allows you to choose whether to restart a container if its not stoppe0d by a user, you can achieve this by using the option `--restart`.
        
        ![restart options](assets/ropts.png)
        
        - When your system boots up, systemd runs the following podman command to start all the containers with the restart policy set to `always`.
            - `/usr/bin/podman start --all --filter restart-policy=always`.
        - The two systemd service files used to restart service, one for rootful and one for rootless:
        → /usr/lib/systemd/system/podman-restart.service
        → /usr/lib/systemd/user/podman-restart.service
        
- **Podman containers as systemd services**
    - Podman has a feature to generate unit files with the best defaults using the command `podman generate systemd`.
        - `podman create -p 8080:80 --name myapp httpd:latest` ⇒ we created a container using the httpd image, now lets generate a unit file off of this container.
            
            ```bash
            $ mkdir -p ~/.config/systemd/user
            $ podman generate systemd myapp > $HOME/.config/systemd/user/myapp.service
            $ systemctl --user daemon-reload #to reload systemd database
            $ systemctl --user start myapp #to start the container
            $ podman ps #notice the container is being runned
            $ curl localhost:8080
            	<html><h1>it works!!</h1></html>
            $ systemctl --user stop myapp #to stop the container
            $ systemctl --user status myapp
            $ cat $HOME/.config/systemd/user/myapp.service
            #notice the ExecStart and ExecStop fileds, using the container ID :
            ExecStart=/usr/bin/podman start <container_ID>
            ExecStop=/usr/bin/podman stop -t 10 <container_ID>
            ```
            
        
        <aside>
        ⚙ One problem with this unit files is that it’s specific to the created container. Hence, you will not be able to hand the unit file to another user and have them run your service on their machine. Luckily, the Podman has the `--new` option which tend to make a more portable systemd unit fie.
        
        </aside>
        
    - `podman generate systemd --new myapp > $HOME/.config/systemd/use/newapp.service` the `--new` flag tells podman to generate units that run,stop, and remove containers.
        - Notice the ExecStart changed to `ExecStart=/usr/bin/podman run --cidfile=%t/%n.ctr-id --cgroups=no-conmon --rm --sdnotify=conmon -d --replace -p 8080:8080 --name myapp`⇒ Podman added additional options to make running under systemd easier.
        - Once you remove the container and the image from your system and tells `systemctl` to start the service, Podman will pull the image and create a new container. This means the newapp.service unit file can be shared with a different user and when they run the service, Podman will pull the image and run the container on their system without them ever creating the container in the first place.
        
        | **Option** | **Command** |
        | --- | --- |
        | With
        --new | ExecStart=/usr/bin/podman run ...--cidfile=%t/%n.ctr-id --cgroups=no-
        ➥ conmon --rm --sdnotify=conmon -d --replace -p 8080:8080 --name
        ➥ myapp [quay.io/rhatdan/myimage](http://quay.io/rhatdan/myimage)
        ExecStop=/usr/bin/podman stop --ignore --cidfile=%t/%n.ctr-id
        ExecStopPost=/usr/bin/podman rm -f --ignore --cidfile=%t/%n
        ➥ .ctr-idType=notify |
        | Without
        --new | ExecStart=/usr/bin/podman start <container_id>
        ExecStop=/usr/bin/podman stop -t 10 <container_id>
        ExecStopPost=/usr/bin/podman stop -t <container_id> |