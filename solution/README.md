# EX188 - Solutions

## 1. Running Simple Containers:

```bash
[chxmxii@kit]$ podman run -dit --name acme-demo-html -v $HOME/workspace/acme-nginx-html/index.html:/usr/share/nginx/html/index.html:Z,U -p 8001:80 [docker.io/library/nginx](http://docker.io/library/nginx)
[chxmxii@kit]$ curl localhost:8001
	Hello World Nginx
```

Note: using the "U" is useless here since there is no need to map the uid.

```bash
[chxmxii@kit]$ ls -lZ workspace/acme-nginx-html/index.html
	rw-r--r--. 1 chxmxii chxmxii system_u:object_r:container_file_t:s0:c338,c861 18 Aug 21 13:17 workspace/acme-nginx-html/index.html
```

---

## 2. Interacting with the Containers:

```bash
[chxmxii@kit]$ podman run -dit --name acme-demo-nginx -p 8002:80 [docker.io/library/nginx](http://docker.io/library/nginx)
[chxmxii@kit] podman ps
[chxmxii@kit acme-nginx-html]$ podman cp html acme-demo-nginx:/usr/share/nginx/html
[chxmxii@kit acme-nginx-html]$ podman cp conf/nginx.conf acme-demo-nginx:/etc/nginx/conf.d/default.conf
[chxmxii@kit acme-nginx-html]$ podman exec acme-demo-nginx nginx -s reload
[chxmxii@kit]$ curl localhost:8002
Welcome to Acme
=> Didn't actually copy the default.conf is empty
```

---

### 3. **Injecting Variables into Containers:**

```bash
[chxmxii@kit]$ podman run -d --name acme_nginx_container_1 -e RESPONSE="Welcome_acme_nginxcontainer_1" -p 8003:8080 quay.io/myacme/welcome
[chxmxii@kit]$ podman run -d --name acme_nginx_container_2 -e RESPONSE="Welcome_acme_nginxcontainer_2" -p 8003:8080 quay.io/myacme/welcome
[chxmxii@kit]$ curl localhost:8003
Welcome_acme_nginxcontainer_1
[chxmxii@kit]$ podman stop 45; podman start 6e
[chxmxii@kit]$ curl localhost:8003
Welcome_acme_nginxcontainer_2
```

---

### 4. **Building Custom Container Image:**

- **amce-db:**
    
    ```docker
    #amce-db-Containerfile
    FROM mariadb:latest
    ARG ACME_MARIADB_DATABASE
    ARG ACME_MARIADB_PASSWORD
    ENV MARIADB_DATABASE=$ACME_MARIADB_DATABASE
    ENV MARIADB_ROOT_PASSWORD=$ACME_MARIADB_PASSWORD
    ```
    
    ```bash
    [chxmxii@kit]$ podman build --build-arg ACME_MARIADB_DATABASE="acme" --build-arg ACME_MARIADB_PASSWORD="acme" -t acme-mariadb:latest .
    [chxmxii@kit]$ podman push acme-mariadb:latest acme:5000/acme-mariadb:latest
    [chxmxii@kit]$ podman inspect --format "{{.Config.Env}}" acme-mariadb:latest
    [PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin GOSU_VERSION=1.14 LANG=C.UTF-8 MARIADB_VERSION=1:11.0.3+maria~ubu2204 MARIADB_DATABASE=acme MARIADB_ROOT_PASSWORD=acme]
    [chxmxii@kit]$ podman run -d --name mariadb--test acme-mariadb:latest
    [chxmxii@kit]$ podman ps
    > exists
    ```
    
- **amce-db-exporter:**
    
    ```docker
    FROM mariadb:latest
    COPY scripts/export.sh /scripts
    WORKDIR /scripts
    CMD ["/bin/bash","/export.sh"]
    ```
    
    ```bash
    [chxmxii@kit]$ podman build -t acme-mariadb-export:latest .
    [chxmxii@kit]$ podman run -d --name mariadb-export-test acme-mariadb-export:latest
    Error: workdir "/scripts" exists on container e5dd920217b16eee1a355a584cda6afafbade0d1fbcb9ee81d7bbf0d07349e1b, but is not a directory
    ```
    

---

### 5. **Multi-container Deployment:**

```bash
[chxmxii@kit ~]$ podman network create acme-wp-net
acme-wp-net
[chxmxii@kit ~]$ podman volume create acme-wp-backend
acme-wp-backend
[chxmxii@kit ~]$ podman volume create acme-wp-app
acme-wp-app
[chxmxii@kit ~]$ podman volume create acme-wordpress_data
acme-wordpress_data
```

```yaml
Version: 3.7
volumes:
  acme-wp-backend:
  acme-wp-app:
  acme-wordpress-data:
services:
    acme-wp-db:
      image: quay.io/myacme/mariadb:latest
      container_name: mariadb 
      networks:
        - acme-wp-net
      volumes:
        - acme-wp-backend:/bitnami/mariadb:Z,U
      environment:
        - MARIADB_USER=acme_wordpress
        - MARIADB_PASSWORD=acme
        - MARIADB_DATABASE=acme_wordpress
        - MARIADB_ROOT_PASSWORD=acme
      restart: always
    acme-nginx:
      image: quay.io/myacme/nginx
      container_name: acme-wp-app
      networks:
        - acme-wp-net
      volumes:
        - acme-wp-app:/etc/nginx:Z,U
      ports:
        - "8080:8080"
      restart: always
    acme-wp-nginx:
      image: quay.io/myacme/wordpress:latest
      container_name: acme-wordpress
      networks:
        - acme-wp-net
      volumes:
        - acme-wordpress-data:/bitnamli/wordpress:Z,U
      ports:
        - "8004:8080"
        - "8443:8443"
      environment:
        - WORDPRESS_DATABASE_USER=acme_wordpress 
        - WORDPRESS_DATABASE_PASSWORD=acme 
        - WORDPRESS_DATABASE_NAME=acme_wordpress
      restart: always
networks:
  acme-wp-net:
```

```bash
[chxmxii@kit]$ podman-compose up -d
```

---

### 6. **Trouble-shoot Multi-container Stack:**

```bash
[chxmxii@kit]$ podman network create acme-troubleshoot
[chxmxii@kit]$ podman volume create acme-wp-backend-ts
[chxmxii@kit]$ podman run -d --name mariadb-ts --network=acme-troubleshoot -v acme-wp-backend-ts:/usr/share/mysql:Z,U quay.io/myacme/mariadb:latest
[chxmxii@kit]$ podman run -d --name acme-nginx-ts --network=acme-troubleshoot quay.io/myacme/nginx
[chxmxii@kit]$ podman run -d --name acme-wordpress-ts --restart=always --network=acme-troubleshoot quay.io/myacme/wordpress:latest
```