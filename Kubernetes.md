Kubernetes – High Level
=======================

What is Kubernetes
------------------

As containerization technology has become more prevalent there became a need for some way to scale large amounts of containers very quickly and to do so in a way that existing services are still available.  Kubernetes is defined as a “container orchestration” tool and is the industry standard for deploying and managing large numbers of containers. The logo for Kubernetes is a wheel with 7 spokes which is both a nautical reference to docker as well as a reference to the original name of the project “sevens” based on 7 of 9 from Star Trek.

Server/Client Topology
----------------------

The technology itself is a distributed system of nodes that is managed by a small number of control nodes. The data plane is represented by “worker nodes” and the control and management plane is represented by “master nodes”. There must be at least 1 master node and 1 worker node in a Kubernetes cluster but they can be on the same physical machine.

The master node has servers running in containers that handle the “scheduling”(load balancing), “Control Management”(status,metrics), and an API server which is the only way for data to come in or out of the cluster. A snapshot of all configuration and node states is also stored on the master nodes for restoration and failover.

Worker Node Design
------------------

Worker nodes are the physical machine states that you are using. For me this was bare metal servers but for most people this is likely your cloud machines. Worker nodes are pointed at the master node via a “kubelet” that is an agent to create a session with the master.

Containers/services are run inside “Pods” which is an abstraction of the container itself and a similar to a placeholder for the container in the cluster’s eyes. Kubernetes will interact with pods and not the container itself (as opposed to something like docker).

Each pod in the cluster is given a unique internal ip (inaccessible outside the cluster) that all pods use to communicate with each other and the master. If a pod dies for any reason a new pod is generated and a new ip is assigned. To prevent downtime pods typically use a “service” tag which exists independent of the pods lifetime therefore removing the need to get a new one during pod lifecycles.

Container/Pod Deployment
------------------------

To deploy pods in a cluster a master node needs a “config file” for the nodes somehow. The three ways are interact with a master node are through the kubernetes dashboard, with an API call, or with a CLI.  Personally I have only used the CLI method to upload configuration but find it to be the best solution for me. Like docker there are various ways you can get container specifics to be implemented.

Docker-Compose is my preferred way to do anything in Docker. The Kubernetes equivilent is called a “Helm Chart” and is where you control your ENV, Volumes, Services, Ports, and how many pods to deploy. The master nodes will check this YAML (or JSON) file regularly and if there is a discrepancy (config calls for 100 instances but only 99) the master node will make any changes to the cluster to get the “actual state” to the “desired state”.

My Kubernetes Implementation
----------------------------

There are many different iterations of Kubernetes you can deploy (mikrok8s is endorsed by canonical/ubuntu) I used k3s as there was a very good NetworkChuck youtube video on the process.

I used Ubuntu 20.04 64bit on 5 Raspberry pi’s that I had in my rack. I started by using SSH to get into all of them:

1.  cd into /boot/ directory and edit cmdline.txt
2.  insert anywhere: cgroup\_memory=1 cgroup\_enable=memory
3.  Save and exit that file
4.  enable old iptables: sudo iptables -F sudo update-alternatives –set iptables /usr/sbin/iptables-legacy sudo update-alternatives –set ip6tables /usr/sbin/ip6tables-legacy
5.  reboot device
6.  on master: curl -sfL https://get.k3s.io | K3S\_KUBECONFIG\_MODE=”644″ sh -s –
7.  on master: sudo cat /var/lib/rancher/k3s/server/node-token
8.   copy token from step 7 and on all worker nodes: curl -sfL https://get.k3s.io | K3S\_TOKEN=”YOURTOKEN” K3S\_URL=”https://\[your server\]:6443″ K3S\_NODE\_NAME=”servername” sh –

From here I used Kubectl to run some ARM Helm Charts I found. When I understand the volume stores and database redundency more I hope to have more critical stuff in my home lab running off Kubernetes. For now I am not using it for anything production oriented.
