[![Build Status](http://34.125.103.67:8080/buildStatus/icon?job=instavote%2Fworker-test&subject=worker-test )](http://34.125.103.67:8080/job/instavote/job/worker-test/)
[![Build Status](http://34.125.103.67:8080/buildStatus/icon?job=instavote%2Fworker-build&subject=worker-build )](http://34.125.103.67:8080/job/instavote/job/worker-build/) 

Jenkins is runnind in a GCP VM instance. Has one node configured (GCP VM instance) and it does not run jobs on the built-in node. 

### CI
Main branch is protected by two checks.
* At least one review approved
* Jenkins continuous-integration/jenkins/pr-head must pass

Since this is mono repository each service has it's own Jenkinsfile in their directory. There is also a Jenkinsfile for the entire repository.

Code is scanned by Sonar.

Jenkins sends build result to Slack via the Slack plugin.

### CD
The last stage of the master pipeline is to trigger ArgoCD. This stage modifies the lastest version in the [GitOps repo](https://github.com/pipilacha/instavote-deploy) Kubernetes declarative files. ArgoCD will pickup change and synchronize our current configuration to our new version. This is deployed to a GCP Kubernetes cluster.

Example Voting App
=========

Getting started
---------------

Download [Docker](https://www.docker.com/products/overview). If you are on Mac or Windows, [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/). If you're using [Docker for Windows](https://docs.docker.com/docker-for-windows/) on Windows 10 pro or later, you must also [switch to Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

Run in this directory:
```
docker-compose up
```
The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:
```
docker swarm init
```
Once you have your swarm, in this directory run:
```
docker stack deploy --compose-file docker-stack.yml vote
```

Run the app in Kubernetes
-------------------------

The folder k8s-specifications contains the yaml specifications of the Voting App's services.

Run the following command to create the deployments and services objects:
```
$ kubectl create -f k8s-specifications/
deployment "db" created
service "db" created
deployment "redis" created
service "redis" created
deployment "result" created
service "result" created
deployment "vote" created
service "vote" created
deployment "worker" created
```

The vote interface is then available on port 31000 on each host of the cluster, the result one is available on port 31001.

Architecture
-----

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time


Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.
