# CHAPTER 4 - Working with Pods

<aside>
<img src="https://www.notion.so/icons/report_lightgray.svg" alt="https://www.notion.so/icons/report_lightgray.svg" width="40px" /> **Pods** are a way of grouping together into more comples applicaitons, sharing namespaces, and sharing resource constraints.
**Pods** share most of the options container use, and when you add a container to a pod, it shares these options with all containers in the pod.

</aside>

- Why pods?
    - A big advantage of using pods is that you can manage them as discrete units. Starting a
    pod starts all of the containers within it, and stopping the pod stops all of the containers
    - Architecture
        - Pods consist of **infra container, init container, primary container and sidecar.**
            - **infra container** ⇒ Its only purpose is to hold open the namespaces and cgroups, while containers come and go. However, each pod will have a different infra container.
            - **init container** ⇒ these containers run before the primary containers in the pods are executed. Consists of 2 classes **Once ⇒ Only runs the first time the pod is created / Alwaus ⇒ Runs everytime the pod is started.**
            e.g. a database initialization on a volume. this would allow the primary container to use the dataase.
            - **sidecar containe**r ⇒ used to handle other job, such as modfying the content of a file.

### Creating a Pod

- `podman pod create -p 8080:8080 --name mypod -v .html:/var/www/html:z`
    
    ![Untitled](Untitled%2011.png)
    

### Adding a container to a Pod

- `podman create --pod mypod --name myapp quary.io/chxmxii/myimage`
    
    ![Untitled](Untitled%2012.png)
    
- At this point, Podman examines the infra container, mounts the /var/www/html volume, and joins the namespaces when it launches the sidecar container.

### Starting a Pod

- `podman pod start mypod` will start the pod.
- some noteable args:
    - `--all` ⇒ starts all pods.
    - `--latest` ⇒ start the last pod created.

### Stopping a Pod

- `podman pod stop` mypod will stop the pod for you.
- Some noteable args to use with the stop command:
    - `--all` ⇒ stop all pods.
    - `--latest` ⇒ stop the most recently started pod.
    - `--timeout` ⇒ set the timeout when attempting to stop the containers within a pod.

### Listing Pods

- `podman pod list`
- Some noteable options:
    - `--ctr*` ⇒ This tells podman to list container information within pods.
    - `--format` ⇒ Tells podman to change the output of pods.

### Removing Pods

- A good practice is to list all the containers on the system.
- `podman ps --all --format “{{.ID}} {{.Image}} {{.Pod}}”`
- `podman pod rm mypod`

### Summary

![Untitled](Untitled%2013.png)