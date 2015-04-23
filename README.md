# docker-compose-joc
Docker Compose based Jenkins Operations Center by CloudBees environment.

Uses [Docker Compose](https://docs.docker.com/compose/) to build a Jenkins Operations Center by CloudBees environment using Docker containers.

Mostly specific to Mac OS X but should work on Windows and Linux as well.

####Base includes:
- [dnsdock](https://github.com/tonistiigi/dnsdock) providing DNS for docker containers and exposed to Mac OS X
- JOC by CloudBees (non HA) - availabe at http://joc.demo.docker:8080
  - JNLP port: `4001`
- Client Master api-team (non HA): http://apiteam.demo.docker:8080
- Client Master (HA) mobile-team: http://mobileteam.proxy.docker
  - http://mobileteam1.demo.docker:8080
  - http://mobileteam2.demo.docker:8080
- HA Proxy - stats available at: http://mobileteam.proxy.docker:9000

###Instructions
- install [boot2docker](https://github.com/boot2docker/osx-installer/releases/tag/v1.5.0)
- install [Docker Compose](https://docs.docker.com/compose/install/)
- run boot2docker and add `bip` and `dns` docker daemon default options
  - `boot2docker ssh`
  - `vi /var/lib/boot2docker/profile`
  - add the following line and save: `EXTRA_ARGS="-bip=172.17.42.1/24 -dns 172.17.42.1 -dns 8.8.8.8"`
  - exit ssh and restart boot2docker: `boot2docker restart`
  - update the VirtualBox network adapter (vboxnet<x> - number may vary) *Promiscuous Mode* to *Allow All*
- Route traffic from Mac OS X to boot2docker VM IP: `sudo route -n add -net 172.17.0.0 <BOOT2DOCKER_IP>`
  - BOOT2DOCKER_IP retrieved via `boot2docker ip`
- Configure OS X to use dnsdock DNS by creating the file `/etc/resolver/docker` with content of `nameserver 172.17.42.1`
NOTE: If the directory `/etc/resolver` doesn't exist create one `sudo mkdir -p /etc/resolver`
- clone this repo to (that is what the Jenkins `HOME` default setting points to on the Mac: `/path/to/docker-compose-joc/`
- If you wouldl like to have your Jenkins `HOME` directory somewhere else you need to update the `docker-compose.yml` file:
  - Update `/path/to/docker-compose-joc/` under dnsdock -> volumes to point to where you want your Jenkins `HOME` directory.
> NOTE: You could have several different directories configured for different demos and just change this to point to the demo you want to run.
- Run the setup `docker-compose up -d` and `docker-compose logs` to view the logs


###Gotchas
If you are no longer able to access docker container hosts via Mac OS X:
- check that the route is correct: `sudo route -n add -net 172.17.0.0 192.168.59.103`
  - Gateway should be `boot2docker ip`
- make sure you are able to ping the `boot2docker ip` - ex (the IP may vary): `ping 192.168.59.103` from Mac OS X
- check to see that the `ip route` you added, still points to your `boot2docker ip` - `sudo route -n get 172.17.42.1`
- You may have to flush DNS cache - on Yosemite use: `sudo discoveryutil mdnsflushcache`
> Update : 04/22/15 - dockerfile/haproxy image was missing from docker hub. To create that container locally
> Checkout `git clone https://github.com/dockerfile/haproxy.git`
> `cd haproxy`
> `docker build -t dockerfile/haproxy .`
> `docker images`
> Confirm that `dockerfile/haproxy` image exists

###Create a New Demo
- You should probably fork this repo, but not absolutely necessary
- checkout a new branch or tag: `git checkout -b workflow-demo master`
- update `docker-compose.yml` to include whatever additional Docker containers you may need - dnsdock will automatically expose them to Mac OS X, so you could for example create a Jetty container with the environment parameters `DNSDOCK_NAME=staging` and a `DNSDOCK_IMAGE=jetty` and if you didn't change the dnsdock defautls, your new jetty container will be available at http://staging.jettty.docker
- You may keep the base Jenkins joc and apiteam or start completely from scratch by removing the `var` directory. At start up, Jenkins Enterprise and JOC will rebuild the respective working directories from scracth - so this may take some time.

You can have as many branches for different demo scenarios that you can think of...
