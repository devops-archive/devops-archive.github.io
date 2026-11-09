# Container Init

Sometimes, we wish a container would behave a little differently from what it was designed to do.

Say for example:

- Change the default user defined in a Dockerfile without having to rebuild the entire Docker image.
- Install additional packages in an existing Docker image without rebuilding it from the original image's source.
- Sometimes, you just want to take the lazy route and directly manipulate a Docker image rather than going through the entire build process.

Or basically I wanted to hijack the docker image, I wanted to do it sometimes. That's why I use this script.

## The Script

Then you may need "con-init" or `coninit.sh` a.k.a "Container Init"

This is a `coninit.sh` for alpine linux based containers.

``` bash
#!/bin/bash

CUID=1000
CGID=1000
USERNAME=anish

#install dependencies
apk update
apk add shadow sudo bash curl yarn jq

#Create User and Groups and Sudo Access
groupadd -g ${CGID} ${USERNAME}
useradd ${USERNAME} -u ${CUID} -g ${CGID} -m -s /bin/bash
usermod -a -G wheel ${USERNAME}
echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/${USERNAME}

#Run infinitely
tail -f /dev/null
```

If we are using a debian/ubuntu container then the above (for alpine) won't work as expected. So in case of Debian based containers use below.

```bash
#!/bin/bash

CUID=1000
CGID=1000
USERNAME=anish

#install dependencies
apt update
apt install -y passwd sudo bash curl jq vim-tiny git

#Create User and Groups and Sudo Access
groupadd -g ${CGID} ${USERNAME}
useradd ${USERNAME} -u ${CUID} -g ${CGID} -m -s /bin/bash
UID_MAX=500000000 usermod -a -G adm,sudo ${USERNAME}
echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers.d/${USERNAME}

#Run infinitely
tail -f /dev/null
```

## The Compose file

Now to use this `coninit.sh` we wil employ a `docker-compose.yml` file as below:

``` yaml
version: '3.8'
services:
  node-dev:
    image: node:16.16.0-slim
    container_name: node-dev
    working_dir: /srv
    environment:
      - VAR1="Foo"
      - VAR2="Bar"
    volumes:
      - ./con-init.sh:/root/con-init.sh
      - ./app:/srv
    command: sh /root/con-init.sh
```

Create a empty directory named "app" in the same directory which houses `docker-compose.yml` file.

you can put the code files in the newly created "app" directory.

Then do:

```
docker-compose up -d
```

now a new container named `node-dev` with start using the `node:16.16.0-slim` image with the changes we added in the `coninit.sh` file.

Now lets connect to the new container, assuming we created the user as `anish` in the `coninit.sh` file. If you used a different user replace `anish` with the username you used in the below command.

``` bash
docker-compose exec -u anish -it node-dev /bin/bash
```

