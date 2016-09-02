Docker-swarm_demo README
========================


Created by: Donal Cooney

Note: 	This demo is a very basic introduction to the topic named above created 
	purely as a tutorial for myself. Absolutely no gaurantees whatsoever 
	wrt these instructions and/or software.


Docker-swarm_demo Summary:
--------------------------

1. 	SET UP 3 NODES (MANAGER, AGENT1, AGENT2)
2.	INSTALL SWARM ON THE MANAGER NODE
3.	SET UP A DISCOVERY SERVICE  CONSUL. OTHER OPTIONS: ZOOKEEPER, ETCD)
4.	RUN THE SWARM MANAGER
5.	ON OUR 2 AGENTS, RUN SWARM JOIN TO LET THEM JOIN THE CLUSTER
6.	CHECK CLUSTER STATUS





Arch overview
-------------


Manager Mode

-- Docker Engine
---- Container1: Consul                 Key Value Store 
---- Container2: swarm master          	swarm master will connect to the DE's
					once the swarm agents have identified
					themselves and use them as a cluster
					of nodes 

-- docker-client


Agent1

-- Docker Engine
---- Swarm				Swarm agents will comm with the KVS
					(Consul) where thsy will identify 
					themselves
---- 

Agent2

-- Docker Engine
---- Swarm



Instructions:
-------------



	SET UP 3 NODES (MANAGER, AGENT1, AGENT2)



	get clone https://github.com/coonedo/docker-swarm-demo

	C:\Users\coonedo\docker_swarm_project>vagrant up manager

	C:\Users\coonedo\docker_swarm_project>vagrant up agent1

	C:\Users\coonedo\docker_swarm_project>vagrant up agent2

	docker is installed on each node (vagrant runs docker_install.sh)



	INSTALL SWARM ON THE MANAGER NODE


	ssh to manager node (vagrant ssh manager)


vagrant@manager:~$ docker run swarm --help
Unable to find image 'swarm:latest' locally
latest: Pulling from library/swarm
220609e0bc51: Pull complete
b54bf338fe2f: Pull complete
d53aac5750d5: Pull complete
Digest:
sha256:c9e1b4d4e399946c0542accf30f9a73500d6b0b075e152ed1c792214d3509d70
Status: Downloaded newer image for swarm:latest
Usage: swarm [OPTIONS] COMMAND [arg...]

A Docker-native clustering system

Version: 1.2.5 (27968ed)

Options:
  --debug                       debug mode [$DEBUG]
  --log-level, -l "info"        Log level (options: debug, info, warn, error,
fa
tal, panic)
  --experimental                enable experimental features
  --help, -h                    show help
  --version, -v                 print the version

Commands:
  create, c     Create a cluster
  list, l       List nodes in a cluster
  manage, m     Manage a docker cluster
  join, j       Join a docker cluster
  help          Shows a list of commands or help for one command

Run 'swarm COMMAND --help' for more information on a command.
vagrant@manager:~$



	SET UP A DISCOVERY SERVICE  CONSUL. OTHER OPTIONS: ZOOKEEPER, ETCD)


	NOTE: we are setting up 1 consul node here. In prod - set up at least 3

vagrant@manager:~$ docker run -d -p 8500:8500 --name=consul progrium/consul
-se
rver -bootstrap
Unable to find image 'progrium/consul:latest' locally
latest: Pulling from progrium/consul
c862d82a67a2: Pull complete
0e7f3c08384e: Pull complete
0e221e32327a: Pull complete
09a952464e47: Pull complete
60a1b927414d: Pull complete
4c9f46b5ccce: Pull complete
417d86672aa4: Pull complete
b0d47ad24447: Pull complete
fd5300bd53f0: Pull complete
a3ed95caeb02: Pull complete
d023b445076e: Pull complete
ba8851f89e33: Pull complete
5d1cefca2a28: Pull complete
Digest:
sha256:8cc8023462905929df9a79ff67ee435a36848ce7a10f18d6d0faba9306b97274
Status: Downloaded newer image for progrium/consul:latest
5be253b53f2ea311b7e91ff686c09a03a82fc16deb1e6122f927cd8c2383e564
vagrant@manager:~$
vagrant@manager:~$
vagrant@manager:~$




	RUN THE SWARM MANAGER

vagrant@manager:~$ docker run -d -p 4000:4000 swarm manage -H :4000
--advertise 192.168.0.248:4000 consul://192.168.0.248:8500
05f743713154ed2223d2920e6ce9a636f47e93cd16da1c5799592087d8a31e04

	run docker swarm maanger
	-d: run in background mode
	-p map container port 4000 to external port 4000
	swarm manage: create a swarm manger
	-H listen on defualt ip, host 4000
	--advertise: Advertise the ip/host of the docker engine
	consul: consul is our discovery backend



	ON OUR 2 AGENTS, RUN SWARM JOIN TO LET THEM JOIN THE CLUSTER

	vagrant ssh agent1A
	
vagrant@docker-agent1:~$ docker run -d swarm join --advertise=192.168.0.246:237
5 consul://192.168.0.248:8500
Unable to find image 'swarm:latest' locally
latest: Pulling from library/swarm
220609e0bc51: Pull complete
b54bf338fe2f: Pull complete
d53aac5750d5: Pull complete
Digest:
sha256:c9e1b4d4e399946c0542accf30f9a73500d6b0b075e152ed1c792214d3509d70
Status: Downloaded newer image for swarm:latest
0bd14cddf5918a31c45fde14f602237c3b807c033d34d721d2e72455dc7e4d41
vagrant@docker-agent1:~$



	vagrant ssh agent2

vagrant@docker-agent2:~$ docker run -d swarm join --advertise=192.168.0.247:237
5 consul://192.168.0.248:8500
Unable to find image 'swarm:latest' locally
latest: Pulling from library/swarm
220609e0bc51: Pull complete
b54bf338fe2f: Pull complete
d53aac5750d5: Pull complete
Digest:
sha256:c9e1b4d4e399946c0542accf30f9a73500d6b0b075e152ed1c792214d3509d70
Status: Downloaded newer image for swarm:latest
3d695c9d1d19b0270e566a454d75dd1b09e86d028775826d0c6f093afd9f0f38
vagrant@docker-agent2:~$




	CHECK CLUSTER STATUS
	

vagrant@manager:~$ docker -H :4000 info
Containers: 2
 Running: 2
 Paused: 0
 Stopped: 0
Images: 2
Server Version: swarm/1.2.5
Role: primary
Strategy: spread
Filters: health, port, containerslots, dependency, affinity, constraint
Nodes: 2
 docker-agent1: 192.168.0.247:2375
  吾西五 ID: 5Y6G:PD63:7HD6:H555:HVY4:SFKH:KUIH:RTTR:LXJE:K3GE:OHAX:B2NV
  吾西五 Status: Healthy
  吾西五 Containers: 1 (1 Running, 0 Paused, 0 Stopped)
  吾西五 Reserved CPUs: 0 / 1
  吾西五 Reserved Memory: 0 B / 514.5 MiB
  吾西五 Labels: kernelversion=3.13.0-83-generic, operatingsystem=Ubuntu
14.04.4 LTS
 storagedriver=aufs
  吾西五 UpdatedAt: 2016-09-02T17:22:44Z
  吾西五 ServerVersion: 1.12.1
 docker-agent2: 192.168.0.246:2375
  吾西五 ID: 6T2E:AJ3N:DH6W:CGC7:NHX7:E6G2:WLJZ:6IMG:V2KC:LACA:33PA:LUAN
  吾西五 Status: Healthy
  吾西五 Containers: 1 (1 Running, 0 Paused, 0 Stopped)
  吾西五 Reserved CPUs: 0 / 1
  吾西五 Reserved Memory: 0 B / 514.5 MiB
  吾西五 Labels: kernelversion=3.13.0-83-generic, operatingsystem=Ubuntu
14.04.4 LTS
 storagedriver=aufs
  吾西五 UpdatedAt: 2016-09-02T17:22:36Z
  吾西五 ServerVersion: 1.12.1
Plugins:
 Volume:
 Network:
Swarm:
 NodeID:
 Is Manager: false
 Node Address:
Security Options:
Kernel Version: 3.13.0-83-generic
Operating System: linux
Architecture: amd64
CPUs: 2
Total Memory: 1.005 GiB
Name: 05f743713154
Docker Root Dir:
Debug Mode (client): false
Debug Mode (server): false
WARNING: No kernel memory limit support
vagrant@manager:~$






	RUN A CONTAINER ON OUR CLUSTER

	
vagrant@manager:~$ docker -H :4000 run -P -d training/webapp python app.py
46ae89e815cfb255f44e821b6fc12b562affbc56bb8fa80cd1ef35ad5292f01d
vagrant@manager:~$



vagrant@manager:~$ docker -H :4000 ps
CONTAINER ID        IMAGE               COMMAND             CREATED
STATUS              PORTS                           NAMES
46ae89e815cf        training/webapp     "python app.py"     2 minutes ago
Up 2 minutes        192.168.0.246:32768->5000/tcp   docker-agent2/goofy_hopper
vagrant@manager:~$
vagrant@manager:~$
vagrant@manager:~$


	
	
vagrant@manager:~$ curl 192.168.0.246:32768
Hello world!vagrant@manager:~$

	









