---
title: How to deploy nopCommerce to Linux VPS using Docker
date: "2020-02-25T10:40:32.169Z"
description: Simple guide to cover how to deploy nopCommerce to Linux server using Docker
---

[nopCommerce](https://www.nopcommerce.com/) is a ASP.NET Core open-source eCommerce solution. I have recently built a site as my side project and deployed it to Ubuntu VPS offered by [Vultr](https://www.vultr.com/). This post focus on the deployment process not nopCommerce product.

### 📚 Infrastructure and Technologies

- Linux server Ubuntu 18.04 as docker host
- [Docker](https://www.docker.com/)
- [Nginx](https://www.nginx.com/resources/wiki/) as reverse server
- [Let's Encrypt](https://letsencrypt.org/)
- ASP.NET Core, SQL Server

### 🦾 Step by step guide

1. **Connect to VPS with SSH**

Create new user in new server and set up firewall

```shell
# Log in as root user
ssh root@your_server_ip
adduser [username]
usermod -aG sudo sammy

# Allow firewall for ssh connection
ufw allow OpenSSH
```

Install [PuTTY](https://www.putty.org/) if you are working in Windows enviroment. PuTTY comes with PuTTYgen utility which can be used to generate authentication keys. Once keys are generated, we will need to copy public key to **~/.ssh/authorized_keys** in the server, so that connection can be automated. Run below commands if file not exist.

```shell
mkdir ~/.ssh
chmod 0700 ~/.ssh
touch ~/.ssh/authorized_keys
# Paste you public key to the file and save
chmod 0644 ~/.ssh/authorized_keys
```

2. **Install docker**

```shell
# update your existing list of packages
sudo apt update

# add docker repository to APT
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce

# install docker
sudo apt install docker-ce
```

3. **Prepare docker images**

   nopCommerce comes with _Dockerfile_ and _docker-compose.yml_, however it is not suitable for deployment behind a reverse proxy server. The website is now sitting behind nginx, `expose` port to internal network instead of `port` publishing.

_Dockerfile_

```yml
# create the build instance
FROM microsoft/dotnet:2.2-sdk AS build

WORKDIR /src
COPY ./src ./

# restore solution
RUN dotnet restore NopCommerce.sln

WORKDIR /src/Presentation/Nop.Web

# build and publish project
RUN dotnet build Nop.Web.csproj -c Release -o /app
RUN dotnet publish Nop.Web.csproj -c Release -o /app/published

# create the runtime instance
FROM microsoft/dotnet:2.2-aspnetcore-runtime-alpine AS runtime

WORKDIR /app
RUN mkdir bin
RUN mkdir logs

# Temp fix alpine sqlclient connection issue
RUN apk add --no-cache icu-libs
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false

COPY --from=build /app/published .

ENTRYPOINT ["dotnet", "Nop.Web.dll"]
```

_docker-compose.yml_

```yml
version: "3.4"
services:
  nopcommerce_web:
    build: .
    container_name: nopcommerce
    # ports:
    #    - "80:80"
    expose:
      - "5000"
    networks:
      - nopcommerceapp
    depends_on:
      - nopcommerce_database
  nopcommerce_database:
    image: "microsoft/mssql-server-linux"
    container_name: nopcommerce_mssql_server
    volumes:
      - sqldata:/var/opt/mssql
    networks:
      - nopcommerceapp
    environment:
      SA_PASSWORD: "nopCommerce_db_password"
      ACCEPT_EULA: "Y"
      MSSQL_PID: "Express"

volumes:
  nopcommerce_data:

networks:
  nopcommerceapp:
```

Run `docker-compose build` to build the images. The images can be transfered to server through Docker Hub registry or manually save and load. Run `docker push` to publish to Docker Hub and then `docker pull` in hosting server. Alternatively, `docker save` can be used to export docker image and `docker load` to load saved image to different server.

4. **Set up reverse server and Let's Encrypt**

Thanks to [docker nginx proxy companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion), the set up is as easy as running a few scripts. Replace bracket variable with suitable values.

```shell
docker run --detach \
    --name nginx-proxy \
    --publish 80:80 \
    --publish 443:443 \
    --volume /etc/nginx/certs \
    --volume /etc/nginx/vhost.d \
    --volume /usr/share/nginx/html \
    --volume /var/run/docker.sock:/tmp/docker.sock:ro \
    jwilder/nginx-proxy

docker run --detach \
    --name nginx-proxy-letsencrypt \
    --volumes-from nginx-proxy \
    --volume /var/run/docker.sock:/var/run/docker.sock:ro \
    --env "DEFAULT_EMAIL=[admin@admin.com]" \
    jrcs/letsencrypt-nginx-proxy-companion

docker run --detach --expose 5000 \
    --name zerominishop \
	--env "ASPNETCORE_URLS=http://*:5000" \
    --env "VIRTUAL_HOST=[valid-domain-name]" \
    --env "VIRTUAL_PORT=5000" \
    --env "LETSENCRYPT_HOST=[valid-domain-name]" \
    --env "LETSENCRYPT_EMAIL=[admin@admin.com]" \
    [your-image]
```

The final steps are to run images from step 4 witn `docker run` commands.

P.S.
The above process can be further improved by consolidating all images build to _docker-compose.yml_, or even better set up CI\CD pipeline to automate deployment. Maybe another post in the future 📆.
