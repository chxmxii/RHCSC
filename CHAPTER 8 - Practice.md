# CHAPTER 8 - Practice

## Practice I

### Objective:

- Deploy images using Podman - Practice using the FROM , ADD, COPY , RUN , ENV , CMD and ENTRYPOINT Instructions parameters.

### Questions:

- Create a simple web server container by following these steps:
    1. Create a new directory for your project.
    2. In the new directory, create a file called **Containerfile**.
    3. In the Dockerfile, add the following lines:
    
    ```bash
     FROM nginx:latest
     ENV TZ=America/Los_Angeles
     CMD ["nginx", "-g", "daemon off;"]
    
    ```
    
    1. Build an image with the following name **my-web-server**
    2. Run the my-web-server container image using host port 8081 and container port 80. Create the new container with the name **my-web**
    3. Read the index.html file from the my-web-server container
    4. Make a backup of the container's index.html file in the project directory on your host
    5. Create an index.html file on your host and put it in the container. The index.html file should have the following text
    
    ```bash
    <!DOCTYPE html>
    <html lang="en">
    <head>
       <metacharset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>My website</title>
    </head>
    <body>
       <header>
         <h1>Welcome to my website</h1>
       </header>
    
       <nav>
         <ul>
           <li><a href="#">Home</a></li>
           <li><a href="#">About</a></li>
           <li><a href="#">Contact</a></li>
         </ul>
       </nav>
    
       <main>
         <section>
           <h2>About</h2>
           <p>This is a sample page.</p>
         </section>
    
         <section>
           <h2>Contact</h2>
           <p>You can contact me via email or social networks.</p>
         </section>
       </main>
    
       <footer>
         <p>Copyright &copy; 2023. All rights reserved.</p>
       </footer>
    </body>
    </html>
    
    ```
    
    1. Restart the nginx service in the my-web-server container
    2. Verify that the index.html works by Browser and by console

## Practice II

### Objective:

Running containers locally with Podman.

### Questions:

1. **Basics**:
    - Create a container based on the "nginx" image and name it "webserver".
    - Maps port 8080 of the host to port 80 of the container.
    - Runs the container in the background.
    - Verify that the container is running and that the port has been bound correctly.
2. **Container Management:**
    - Create a container based on the image "mysql" and name it "database".
    - Set the environment variable "MYSQL_ROOT_PASSWORD" to a safe value.
    - Allocate a volume to persist the container data on the host.
    - Run the container and confirm that it is running.
    - Verifies that the container has access to persistent data.
3. **Networking:**
    - Create two containers based on the "httpd" image and name them "frontend" and "backend".
    - Create a bridge type network for the containers.
    - Connect both containers to the created network.
    - Verifies that the containers can communicate with each other on the same network.
4. **Image construction:**
    - Create a Containefile in an empty directory.
    - Use a suitable base image and copy your app inside the container.
    - Expose a port on the container.
    - Build the image using Podman and give it an appropriate name.
    - Run a wrapper based on the newly created image and verify that it works correctly.
5. **Storage:**
    - Create a container based on the Docker "registry" image.
    - Mounts a volume on the container to store the log data.
    - Assign a port on the host to access the registry.
    - Upload a custom image to the registry using Podman.
    - Verify that the image has been uploaded correctly and is available for download.

## Practice III

### Objectives

- Create container images in Podman by passing or defining variables during the build of the image itself.

### Questions

- Create a ContainerFile that creates a container image with the following requirements.
    1. Based on centos:7.
    2. During the build, create a user account. "benito" should be the default user.
    3. Read an argument during construction to override the name "benito".
    4. The container must execute "whoami" to display the active user.

**NOTE**: Arguments are NOT passed during container runtime! They are passed during the construction of the container.

- Create two container images using this containerfile, named benito-came:1.0 and elsa-marcela:1.0. When run, they should display "Hello" and "World" respectively.