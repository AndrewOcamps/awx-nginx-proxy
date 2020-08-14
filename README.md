# awx-nginx-proxy
Install and Configure Nginx as a Reverse Proxy for Ansible Tower / AWX

We are going to install a container with Nginx as a reverse proxy and built on top of the secure HTTPS connection.

#### Previous Settings:

Change the awx_web container configuration so that it only receives internal requests from the server

```
PORTS               NAMES
8052/tcp            awx_web
```

It should be as follows:

```
sudo docker ps | grep -i awx_web

CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS               NAMES
167e0fbf0f3c        ansible/awx:14.0.0   "tini -- /bin/sh -..."   45 hours ago        Up 6 hours          8052/tcp            awx_web
```

#### Service installation

Clone the repository on the server where the AWX containers are running

```
git clone https://github.com/AndrewOcamps/awx-nginx-proxy.git
```

In the docker-compose.yml file we change the settings according to need.
For example, specify the certificates to install in the nginx service

```yaml
    volumes:
      - "./files/certs/localhost.crt:/tmp/localhost.crt:ro"
      - "./files/certs/localhost.key:/tmp/localhost.key:ro"
```

Define the connection network so that the nginx container can communicate with the awx_web container

```yaml
    networks:
      - awxcompose_default
```

To find out which network your AWX configuration uses by default, you can launch the following command:

```
sudo docker network list

NETWORK ID          NAME                 DRIVER              SCOPE
6714cf2ca553        awxcompose_default   bridge              local
a3f5b2fa25db        bridge               bridge              local
d3abb6e61e9b        host                 host                local
15dc764ac666        none                 null                local
```

Define the variables with the configuration of your need

 ```yaml
     environment:
      - NGINX_HOST=awx.ocampoge.com
      - NGINX_IP=192.168.33.90
      - AWX_WEB=awx_web:8052
 ```

 * __NGINX_HOST__ => Server domain name
 * __NGINX_IP__ => IP address of the server
 * __AWX_WEB__ => Name of the container used for the web interface
 
If you use SELINUX you must apply the following context configuration for the certificates and the configuration file
 
 ```
 sudo chcon -Rt svirt_sandbox_file_t ./files/
 ```
 
To start the container, launch the following command:
 
 ```
 sudo docker-compose up -d
 
 Starting awx_nginx ... done
 ```
 
It should be as follows:

```
sudo docker ps

CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                                      NAMES
ee73e9535200        nginx:1.19.2         "/docker-entrypoin..."   5 hours ago         Up 15 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   awx_nginx
167e0fbf0f3c        ansible/awx:14.0.0   "tini -- /bin/sh -..."   46 hours ago        Up 6 hours          8052/tcp                                   awx_web
4ea178ed1474        ansible/awx:14.0.0   "tini -- /usr/bin/..."   2 days ago          Up 6 hours          8052/tcp                                   awx_task
be153ae0977e        postgres:10          "docker-entrypoint..."   2 days ago          Up 6 hours          5432/tcp                                   awx_postgres
1b72d3d1c556        redis                "docker-entrypoint..."   2 days ago          Up 27 minutes       6379/tcp                                   awx_redis
```
