# Kubernetes Revision
~~~ 
Please note this is not exhaustive list.
But just pointers that I want to remember so I might have used shortcuts or just jotted down points which I feel are important.

For any references please search keywords in kubernetes documentation or cross verify things on Google :)
~~~

Udemy Course - Mumshad Mannambeth

~~~
-------  3 Certifications -------
1. CKA - Administrator
2. CKAD - Application Developer
3. CKS - Security
~~~


##CKA


### Kubernetes Components 

#### Master 
- etcd
  - DB
  - keyvalue
- kube-api
  - Responsible for communication between rest of the components 
  - All the commands are directed to API server which then takes necessary action
  - only point of interation with etcd DB
- kube-scheduler 
  - Responsible to identify nodes on which pods should be scheduled based on
      - policy
      - constraints
      - (eg - resource avaibility , taints, tolerance, affinity etc)
- kube-controller manager 
  - node controller
  - Replication controller
  - etc

#### Worker
 - kubelet
   - Registers Node
   - Create Pods
   - Monitor node and pods
   - Needs to be installed manually on all the nodes (Works as service)
 - kube-proxy
   - Responsible for all Networking
   - Plays with IPTables (For Service)
   - Service is a virtual Component (No Pods run to make a service. Its just a logical entity)
   
---------------------

### Additional Details 
	
#### ETCD
 - default port 2379
 - etcdctl has 2 version 
   - version 2
   - version 3 
 - commands for both versions are different so make sure which version you are pointing to
 - default is version 2
 - Use env variable to change the version `export ETCDCTL_API=3`
	
#### Kube API
 - Can be run as a container or service 
 - has various different configrable options 
 	- such as TBD
 - Configuration Files can sit at different locations depending on how its installed
   - `cat /etc/kubernetes/manifests/kube-apiserver.yaml`
   - `cat /etc/systemd/system/kube-apiserver.service`
   - `ps -aux | grep kube-apiserver`
	
#### Kube-controller 
 - Can be run as a container or service 
 - has various different configrable options 
 	- such as TBD
 - Configuration Files can sit at different locations depending on how its installed
   - `cat /etc/kubernetes/manifests/kube-controller-manager.yaml`
   - `cat /etc/systemd/system/kube-controller-manager.service`
   - `ps -aux | grep kube-controller-manager`
  
#### Kube-scheduler
 - Identifies nodes to Schedule pods on
 - Has mechanism to first filter the nodes bases on policies and constraints 
 - Then Rank role based on resource avalibilty and then selects appropriate node
 - People can write their own schedular if needed  

----------------

## Kubernetes Defination file 
At highlevel all the kubernetes defination file has 4 required components 

~~~ 
 apiVersion :
 kind: 
 metadata :
 spec:
~~~

All the information on above tag depends on what kind of resource you are going to make. So check the documentation as required.

----------------
#### Replica Sets
 - k8s previously used to have Replication Controllers 
 - As name suggests used to keep eyes on the pods and restart if any pods are killed or dies. To bring pods to desired state
 - Replica Sets should be used over Replication Controller
 - Both are similar in primary functionality just that Replica sets have additional field in configuration called `selector`
 - `selector ` is used to identify pods which will be run or already running by thier lables and then Replica sets starts to monitor them
 - catch here is pods need not be created by Replica set to be monitored but can be existing pods with the tags that `selector is looking for`
 - Replica set adds and checks for `ownerReferences`
   - `metadata.ownerReference`
   - Only if object doesnt have a owner i.e its orphaned then it takes it on the watch list.
   - `that tag` is also tied to garbage collection mechanism (TBD)
 - IDK why this was introduced as it can complicate things if existing pods are running different images! -- NEED TO READ MORE!

#### Namespaces (Also referred as NS in this document)
 - to keep things inside same bucket! 
 - Namespaces share the networking and Storage
 - quota requirment can be implemented on top of Namespace
 	- Can set default minimum cpu and mem for every pod in NS
 	- Upper cap of CPU,MEM for the NS is setup 
 - 4 NS are created when k8s is started
   - default
     - if you dont specify any namespace everything goes in default namespace 
   - kube-system
     - k8s pods go here
   - kube-public
     - tbh idk this  
   - kube-node-lease
     - tbh idk this
 - Default NS is used if nothing is specified.
 - can use `--namespace=dev`  parameter to point to any NS when firing `kubectl` commands  
- Can change the default NS to point to something else by below command
  - `kubectl config set-context $(kubectl config current-context) --namespace=<namespace>`
- Example To connect to DB service across NS
  - `mysql.connect("db-service.dev.svc.cluster.local")`
  - db-service - service name
  - dev - namespace
  - svc is sub domain
  - cluster.local is Default domain name for k8s 

  
#### Services
~~~
Types of services
1. Node Port
2. Cluster IP
3. LoadBalancer
4. Headless (Used with Stateful Sets not in CKA scope)
5. External (Used to point out to external Address not in CKA scope)
~~~ 

##### Node port
- Exposes port over all the VMs so one can access the service if it is inside the same network.
- in k8s config below are paramters used

~~~
 targetPort: (port on which pod is listening)
 port:       (port of the service)
 nodePort:   (port on VM where serive willbe accessed range : 30000-32767)
 selector:   (key value for the pods it should include under the service)
 
 *note this is not in necessary order or parent child linkage only important paramerts that I want to rememeber *
~~~
- only `port` is required feild
  - if no `targetPort` it defaults to `port`
  - if no `nodePort` random port is selects from range `30000-32767`

##### Cluster IP
 - Cluster IP is default service type if no Serivce `type` is specified
 - Used when service just needs to be exposed to other services/consumers inside the k8s cluster

##### Load Balancer
 - exposed service similar to Node Port but adds over a LB if its supported.
 - eg in cloud it will add a loadbalancer
 - if there is no support for LB it will just behave as node Port

## Imperative vs Declarative
Imperative is something that is done step by step one instruction at a time.
EG. - previously you had to guide the cab driver to your destination asking the person to take various turns and identify the landmark.

Similarly if you provide every instruction via CMD to K8s its know as imperative 
sample commands , 

~~~
run
create
replace
delete
edit
scale
expose
etc...
~~~

Delarative is something that you just declare the state and let the engine decide how it wants to reach there. Or what kind of statements it wants to execute

EG. - If you use app like OLA/Uber/Grab  you just need to enter the start and destination rest all is controlled by the app.

Similarly in k8s if you write declarative yaml files and apply it k8s will create things for you. example command is

~~~
apply
~~~

**Note : Never mix imperative and declarative style. And it is adviced to follow Declarative style. (Unless for exam where you want to do quick stuff without maintaining history)**


For  Declarative style it looks at 3 copies before executing any command 

1. Local Copy
2. Live object Defination in K8s
3. Last Applied Configration (Stored in JSON format inside object description)

- If there is change in current local and live object in terms of image etc it applies it and updates the last applied configration.
- If there is modification /deletion on the lables it will first check if previous version had it take appropriate action 
  - EG tag is being deleted. So no present in local but was present in previous version hence delete it
  - For this example if the tag didnt exist in the previous version but was on the live object defination then no action will be taken
  - I dont think this is very important and we need to read more about this! But if I face any issues about this will come and update this section


## Scheduling (Taint/Tolerations, Node Selector and Affinity)
For considering scheduling one needs to think of taints & tolerance, node selector and node affinity/anti-affinity in one box.

Because all of these compliment each other and you need to understand the behaviour well to understand how it works.

- Taint (This is only for/on nodes)
  - To accept pods with certain tolerance.
- Toleration (This is for/on pods)
  - To make pods tolerent to the taints applied on the node
- Node Addinity /Node Selector (This is for/on pods)
  - To make certain pods run on certian nodes 
  - Lables are placed on nodes 


#### Taint / Toleration

##### Taint 
- Tait is for node!  -- Tolerance is for Pod
- If a node is tainted it will only accept pods which has its tolerance 

~~~
 kubectl taint nodes <node-name> key=value:<<TAINT EFFECT>>
~~~

- Taint Effect determines what happes to the pods which do not tolerate the taints.
  - Different type of taint effects are as below
      - **NoSchedule** : Pods are not scheduled if no tolerance
      - **PreferNoSchedule** : Pods are prefered to not be scheduled but no gurantee
      - **NoExecute** : New pods wont be scheduled and existing pods will be deleted
    	  - NoExecute has additional field called `tolerationSeconds` 
    	  - If above field is set then k8s waits for that time before any pods are deleted 
- Think of Taint effect as what can happen if taint is applied before any pods are scheduled and what should happen if node had existing pods and then taint was applied! 

- K8s also uses taints as internal mechanism for kubernetes to work
  - Example master nodes are tainted to run only master components 
  - Whenever a node is unhealthy/unreachable or having any other issues/pressure like cpu,memory,io,pid . K8s internally sets taints to reflect the node behaviour
  - Based on this taints scheduler determines if pods are to be sent to these nodes or not! So in short unhealthy status of node is reflected to scheduler via Taints!

~~~
 kubectl describe node kubemaster | grep taint
 
 # o/p should be something like>> node-role.kubernetes.io/master:NoSchedule
~~~

- Adding and removing taints

~~~
kubectl taint nodes node1 key1=value1:NoSchedule


# To remove the taint added by the command above, you can run:

kubectl taint nodes node1 key1=value1:NoSchedule-
~~~

##### Toleration
- Pods should be attached tolerance so they can run on nodes which has same taint.
- You specify a toleration for a pod in the PodSpec as below

~~~
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
~~~

- Different kind of operator
  - Equal
  - Exists
   
~~~
The default value for operator is Equal.

An empty key with operator Exists matches all keys, values and effects which means this will tolerate everything.

An empty effect matches all effects with key
~~~

**For more details and specific usecases or premutations and combinations of how it works refer documentation its good [taints and toleration] (
https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)**

#### Node Selector
- If you want to schedule a particular pod on particular/group of nodes then node selector is used
- nodeSelector is the simplest recommended form of node selection constraint. `nodeSelector` is a field of PodSpec

~~~
 nodeSelector:
   <<node_label>>
~~~

- to label a node
  - `kubectl label nodes <node-name> <label_key>=<label_value>`
- **Limitations of node Selector** 
  - Only Single Lable can be used
  - If you want advance logics like "Run this pod on Large or Medium Nodes" or "Run this pod on nodes which is not Small" 
  - Node selector wont be able to do that. You need to use `Node Affinity` 


#### Node Affinity  (TBD Pod Affinity/AntiAffinity)

1. The affinity/anti-affinity language is more expressive. The language offers more matching rules besides exact matches created with a logical AND operation

2. You can indicate that the rule is "soft"/"preference" rather than a hard requirement, so if the scheduler can't satisfy it, the pod will still be scheduled

3. You can constrain against labels on other pods running on the node (or other topological domain), rather than against labels on the node itself, which allows rules about which pods can and cannot be co-located (TBD)

~~~
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
~~~

- Importat things to look at in above defination is the long expression (Type of Node Affinity) and Operator
- Operator (Types)
	- In
	- NotIn
	- Exists
	- DoesNotExist
	- Gt
	- Lt

- Types of node Affinity
	- `requiredDuringSchedulingIgnoredDuringExecution`
	- `preferredDuringSchedulingIgnoredDuringExecution`
	- Future
		- `requiredDuringSchedulingRequiredDuringExecution ` 	
- What happens if there are no labels or lables are changed at certain later stage ?
  - Type of node Affinity determines this behaviour 

#### Daemon Sets
- kind `DaemonSet`
- `kubectl get daemonsets`
- Before k8s 1.12 k8s used to use `nodeName` in spec field for Daemon Set to work
- Post 1.12 it uses `nodeAffinity` rules internally to achieve Daemon Set behaviour

#### Static Pods
- pods can be run on node without interacting with k8s api-server. if you are setting up k8s using pods how is it started and managed if no k8s exists? or some issue is there and cluster is not up. (Read api-server is down)
- Static pods can run on nodes directly via kubelet they arent part of k8s directly but they are mirrored to k8s-api so you can see them when you run commands
- These pods arent governed by api-server but by individual `kubelet` on the respect nodes
- Static pods are created from k8s pod spec which will reside on the node
- There are 2 approach to provide the location of these files to `kubelet service`
	- path to the folder is provided by parameter `--pod-manifest-path` which running the kubelet
	- Usually the path defaults to `/etc/kubernetes/manifests`
	- Another approach is to set `--config=kubeconfig.yaml`
	- `kubeconfig.yaml` --> `staticPodPath: /etc/kubernetes/manifests`
- `ps -aux | grep kubelet | grep config`
- If a static pod fails kubelet will automatically restart it
- Static pod will normally have `node name` suffix attached at the end - Easy trick to identify static pods

#### Multiple Schedulers
- K8s is very flexible / extinsible you can have your own scheduler based on any of your custom logic
- multiple schedulers can run at a time on k8s
- Applications can be scheduled via default schedular and few apps can be configured to schedule pointing to custom schedular
- in pod `spec` -- `schedulerName` property is present to point to specific schedular
- To check which schedular the pod was used to run `kubectl get events` or MAYBE check the events of the pod? need to check the later part
- If there is scheduler available you can use `nodeName` property on pod spec and make the pods running


## Networking for Kubernetes

- Network Interface
	- Physical or logical entity on VM which connects it to network
	- Example : eth0, docker0 etc
	- One Network interface can have 'N' no of IP's and that too of different subnets
- Switch
	- Used to connect devices in same Network
- Router
	- Used to connect 2 different Networks (Imagine your house router)
- Gateway
	- Way to tell a particular device where the router is
	- Think this in term of the node/VM
	- Its a configuration which points to Router
- DNS
- Network Namespaces
- Docker Networking

##### Understanding Linux/VM level routing

- Read about "ip" command this is prefered over ifconfig
- Read about "route" / "ip route" command
	- These commands help to create routing tables on the machine
	- Good Youtube video to understand this [Link] (https://www.youtube.com/watch?v=c4rfWsV4H-I&ab_channel=StevenGordon)
	- Cheat sheet to understand the commands [Link] (https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf)

~~~
  ip link			: List and modify interfaces on host
  ip addr			: to see Ip assigned to interfaces
  ip addr add		: set Ip on interfaces

  ip route		: see route table
  ip route add	: add entry in route table
~~~

- One network interface can forward packets to another
	- Use case: Router is connected to many networks that is how it achieves it
	- Modify below file for pack forward

~~~
  cat /proc/sys/net/ipv4/ip_forward
  # >> 0  -- No Forward
  # >> 1  -- Forward

~~~

- Above file or commands are not persistent across reboot you need to modify specific files to make these settings persist over reboot
- File for ip forward peristance is `/etc/sysctl.conf`

~~~
Entry in above file should be

...
  net.ipv4.ip_forward = 1
...

~~~


##### DNS 

- Resolving Name to Ip address
- Each host has `/etc/hosts` file for local information to do DNS resolution
 
~~~
 cat /etc/hosts
 192.168.1.155 test
 192.168.1.11  db
~~~

- But if you have lot of machines and lot of DNS names its best to have your own DNS server
- To point to internal dns server you can modify `/etc/resolv.conf`

~~~
cat /etc/resolv.conf

nameserver 192.168.1.100					<-- points to DNS server
search mycompany.com prod.mycompany.com <-- special type to append domain Name
~~~

- So even if you are doing `ping db` it will modify it to `ping.mycompany.com`
- i.e. domain name is appended with `search` tag

- By default `/etc/hosts` has preference over `DNS server`
- This can be verified or modified in the file `/etc/nsswitch.conf`

~~~
cat /etc/nsswitch.conf

...
passwd:   files nis
group:    files nis

hosts:    files dns myhostname    
...
~~~

- Host information first comes from /etc/hosts (files), then a DNS server (dns), and if neither of those work, at least a fallback of "myhostname" so that the local machine has some name.

- One can host their own DNS server - example using Core DNS

### Network NameSpaces! (Referred to as NNS in section below)

- You can divide Network into different namespaces similar to the way docker does with process
- In docker container you are only able to see the process of container and not that of host.
- However on host you can see the process running inside the docker
- Linux has way to implement network namespaces
-  To list down current NNS  `ip netns`
-  To create new NNS 

~~~
ip netns add red
ip netns add blue

>> ip netns 
	 red
	 blue
~~~

- Network NS dont have any network interfaces provisioned by default except for the loopback 
- To check for Network interface run `ip link` on host so you will see eth0, docker0 etc etc
- But if you run the same inside the NNS you will only see `lo`
- You can exec in NNS similar as we do in docker by below command where `ip netns exec` means exec `red` is the name of NNS and `ip link ` is the command that we want to run inside it

~~~
ip netns exec red ip link
# >> lo
~~~

- To talk between the NNS you need `Virtual cable`
- To create a cable run command

~~~~
ip link add veth-red type veth peer name veth-blue
~~~~

- The above command says `create a cable of type veth` with `network interface veth-red` connected to `network interface veth-bule`
- Now you need to connect respectvie network interface to NNS

~~~
ip link set veth-red netns red
ip link set veth-blue netns blue

~~~

- Now we need to assign IP to both the NIC

~~~
ip netns exec red addr add 192.168.15.1 dev veth-red
ip netns exec blue addr add 192.168.15.2 dev veth-blue
~~~

- Now we need start the link / network interface in each NS
- `ip -n`  is short form for `ip netns exec`
~~~
ip -n red link set veth-red up
ip -n red link set veth-blue up
~~~

- Try to ping from red NS to blue NS it should work
- `ip -n red ping 192.168.15.2`  should work

**But notw the challange is you can keep on creating these virtual cables and adding it to different NNS because there are be 100's of them on same host hence enters Virtual BRIDGE**

#### Bridge -  (Virtual Switch)
- Linux Bridge - native way to create virtual switch
- Other option is Open Vswitch
- Command to create Virtual bridge 

~~~
ip link add v-net-0 type bridge
ip link set dev v-net-0 up 
~~~

- For the host machine Bridge is just like any other network interface. can be listed by the command `ip link`
- But for NNS it can act like `Switch`
- Connect the NNS to the `bridge`

~~~
ip link add veth-red type veth peer name veth-red-br
ip link add veth-blue type veth peer name veth-blue-br
~~~

- Now link the respective new NIC to NNS or bridge

~~~
ip link set veth-red netns red
ip link set veth-red-br netns v-net-0

ip link set veth-blue netns blue
ip link set veth-blue-br netns v-net-0
~~~

- Assign the ip address to the veth-red and blue and UP them to test the ping

~~~
ip -n red addr add 192.168.15.1 dev veth-red
ip -n blue addr add 192.168.15.2 dev veth-blue

ip -n red link set veth-red up
ip -n red link set veth-blue up
~~~

- [TBD ] Some additional pointers to read about IPtables, NAT and Masquerading to make request go outside from NNS via router to internet and back
- NAT : Network Address Translation generally involves "re-writing the source and/or destination addresses of IP packets as they pass through a router or firewall"
- Seems to be good cumulative guide for nat [link] (https://www.karlrupp.net/en/computer/nat_tutorial)



### Docker Networking
- Docker used above Linux Networking Namespace when creating a network over the host
- The bridge created by docker on the host is called `docker0`
- And assigns a private Ip to it which can be seen when you execute below command on the host (Range : 172...)

~~~
ip link
ip addr
~~~

- Docker uses port forwarding in IPtables to make host port talk to container port (NNS)

~~~
iptables \
	-t nat \
	-A Docker \
	-j DNAT \
	--dport 8080 \
	--to-destination 172.17.0.3:80 
~~~ 

- you can see the IPTABLE rules via command

~~~
iptables -nvL -t nat
~~~


### CNI 

- Standard to solve networking challanges with containers
- Gives different standards for plugins which will help runtime to invoke those 

#### Viewing CNI option on running kubelet

~~~
ps -aux | grep kubelet

# >> --cni-bin-dir=/opt/cni/bin
# >> --cni-conf-dir=/etc/cni/net.d
# >> --network-plugin=cni
~~~

### Service Networking

- Whenever there is a new pod to be created kubelet listens to API-server and creates a new pod and invokes CNI plugin to set the networking
- Similarly when ever there is a new service to be created kube-proxy listens to API-server and creates a service (Service is logical entity)
- 3 kind of thing can be used to created a service
	- userspace
	- iptables (default)
	- ipvs
	- to check what kind of service is being created you can look at the kube-proxy logs
- to view iptable rules for a service run below command on host machine

~~~
	iptables -L -t net | grep db-service
~~~

- Services has different IP range then the pods. They should not overlap
- Generall this range is 10.x.x.x but can be configured with kube-api-server option `--service-cluster-ip-range`
- To view the range `ps -aux | grep kube-api-server`
- If its running as container `cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range`
- **Strange why is this in api-server and not in kube-proxy or CNI (weave) setting?**
	- it is in api-server because service is responsibility of kube-proxy and not CNI(weave) that is only responsible for pod networking over the host 


### DNS in Kubernetes

Hostname | Namespace | Type | Root | IP address 
--- | --- | --- | --- |--- |--- |--- |--- |--- |--- |--- |---
web-service  | apps | svc | cluster.local | 10.107.37.188
10-244-2-5 | apps | pod | cluster.local | 10.244.2.5
10-244-1-5 | default | pod | cluster.local | 10.244.1.5

- to access a service across namespace you can use one of the following method

~~~
	curl http://web-service.apps
	curl http://web-service.apps.svc
	curl http://web-service.apps.svc.cluster.local
~~~

#### Core DNS - default DNS since kubernetes 1.12
- k8s  creates 2 pods of coredns by default
- coredns config file is located at `/etc/coredns/Corefile` and passed as configmap
	- it has k8s specific plugin inside 
	- top level domain name of cluster is specified here `cluster.local` 
	- has another configuration `pods insecure`
		- you can tweak this to give pod `-` ip in the dns records
		- **whats the use case for above?** 
- For every pod their `/etc/resolve.conf` file is modified by kubelet to add nameserver entry and search entry
- To check the IP address of coredns to be pointed
	- `kubectl get service -n kube-system`
	- coredns is run as service names `kube-dns` over cluster with type cluster ip
	- `kubectl get svc -n kube-system`

	
~~~
nameserver  10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
~~~

- Only svc entries are added to that file not for pods
- To reach pod you have to write the complete domain name (provided setting for dns is done to map pod ip to pod `-` ip)

~~~
ping 10-244-2-5.default.pod.cluster.local
~~~

### How entire pod-to-pod (same node), pod-to-pod (different node), pod to service works! 

- After reading all the above pointers its good to go through this [link] (https://sookocheff.com/post/kubernetes/understanding-kubernetes-networking-model/) to understand how it all fits in single picture

- **My mental model of how it works**
	- There are 3 set of IP ranges
		- For all the nodes to talk to each other (ex 192.168.1.0/16, 192.168.2.0/16)
		- For the linux bridge on each host for pods (ex 172.10.x.x )
			- this is responsible for all NNS to communicate to each other on same host
			- So **host 1** can have 172.10.1.1, 172.10.1.2 etc (and they can communicate to each other directly because of bridge)
			- **host 2** can have  172.10.2.11, 172.10.1.12 etc (and they can communicate to each other directly because of bridge) 
			- Now we insert **CNI** plugin which sits at the host level and tells it if there is a pod ip which host it should redirect it to
		- For Services (ex 10.x.x.x)
			-  kube-proxy over here has modified the host IP tables using native linux things like  `(Iptables - netfilter, conntrack) ` and modifies the service IP to pod ip directly
`iptables is a user-space program providing a table-based system for defining rules for manipulating and transforming packets using the netfilter framework.`

`Kubernetes includes a second option for in-cluster load balancing: IPVS. IPVS (IP Virtual Server) is also built on top of netfilter and implements transport-layer load balancing as part of the Linux kernel`

~~~
When routing a packet between a Pod and Service, the journey begins in the same way as before. 

The packet first leaves the Pod through the eth0 interface attached to the Pod’s network namespace (1). 

Then it travels through the virtual Ethernet device to the bridge (2). 

The ARP protocol running on the bridge does not know about the Service and so it transfers the packet out through the default route — eth0 (3). 

Here, something different happens. Before being accepted at eth0, the packet is filtered through iptables. 

After receiving the packet, iptables uses the rules installed on the Node by kube-proxy in response to Service or Pod events to rewrite the destination of the packet from the Service IP to a specific Pod IP (4). 

The packet is now destined to reach Pod 4 rather than the Service’s virtual IP. The Linux kernel’s conntrack utility is leveraged by iptables to remember the Pod choice that was made so future traffic is routed to the same Pod (barring any scaling events). 

In essence, iptables has done in-cluster load balancing directly on the Node. Traffic then flows to the Pod using the Pod-to-Pod routing we’ve already examined (5).
~~~

### Ingress

- If you have multiple applications which needs to server multiple customer base you will ideally expose them via your cluster to outside world with service type `nodePort` or `Load Balancer`
- But the challange with both of them is you will either have to expose lot of LoadBalancers to maintain it
- Also if you want to have secured connection `read ssl` you will have challange to execute it
- Hence k8s introduced Ingress to solve this. Ingress can be manupliated/modified as native k8s resources 
- By default k8s doesnt comes with Ingress. you will have to set it yourself.
- 2 components to Ingress
	- Ingress controller
		- Deploy solution like Nginx, HAproxy, countor, go-traffic etc 
	- Ingress Resources
		- k8s native way to introduce ingress rules
		- `kubectl get ingress`

- 2 level of ingress resource rule (to point to appropriate service)
	- Routing based on domain name (`Rule` in ingress file spec)
		- `www.mycompany.com`, `www.payments.mycompany.com` `www.api.mycompany.com`  etc
	- Sub section to above is routing based on path (`Path` in ingress file spec)
		- `/watch` `/checkout` etc etc    
- Note Ingress has to be exposed out side can be done as `nodePort` or `LoadBalancer` but this is onetime activity
- Note `Default backend` service needs to be setup to show good message if routes doesnt match any of the existing rules in Ingress resources
	- name of this service can be found whenever you describe any ingress resource
	- `kubectl describe ingress ingress-wear-watch` 



### Application Lifecycle Managment

- Rolling Updates
- Rollbacks
- Env Variables for container
- Config Maps
- Secrets
- Scale Applications
- Multi Container Pods
- Init Containers 
- Self Healing Applications


#### Rolling Updates
- If a deployment is being update in rolling update fashion. k8s creates a new replica set and then use one pod at a time to start and stop between the old and new replica set.
- Last version of replica set is maintained with "0" pods running so in case of rollback you can easily move back
- Commands to see rollout status and history and rolling back

~~~
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
~~~

- `kubectl describe deployment/myapp-deployment` command will give details of deployment `StratergyType` "Recreate of RollingUpdate(Default)" and `Events` section shows how it was into effect


#### Startup arguments for Docker containers and Kubernetes

##### Docker 

- CMD
- Entrypoint

- `CMD` and `Entrypoint` both takes input as shell command or JSON format.
	- JSON format should be preferred
	- For JSON first argument is always a executable
- `CMD` is default command. but if you have arguments supplied while running the image `CMD` gets overridden completely 
- If you have Entrypoint then any arguments which is supplied to container while running are appened to entrypoint
- If no arguments are provided then entrypoint might fail depending on what you are running
- You can have a combination of `CMD` and `Entrypoint` for things to work if no parameters are provided while running `CMD ` provides the default ones!
- You can also override entrypoint by specifing `--entrypoint` argument while running the image

##### Kubernetes

- As you have seen the CMD and Entrypoint in above section k8s config file has `args` and `command` arguments which match those
- `CMD` --> `args`
- `Entrypoint` --> `command`

~~~
apiVersion : v1
kind : Pod
metadata:
	name : ubuntu-sleeper-pod
spec:
	containers:
		- name: ubuntu-sleeper
		  image: ubuntu-sleeper 
		  command: ["sleep2.0"]
		  args: ["10"]
~~~

#### Config Map

- Way to send env variables to pods

- To invoke `Config Maps` or `secrets` in pods there are 3 options
	- `envFrom` - apps in all the values to pod
	- `env` - You can pick and chose which variables you want and can change their Key
	- `volumes` - each attribute will be mounted as file with its value(key) inside  



### Cluster Upgrade

- k8s components versioning
- No other component can be higher then the API version (consider X) (except kubectl)
- Controller manager and Kube Scheduler can be of versions (X, X-1)
- kubelet and kubeproxy (Worker nodes) can be of version (X,X-1, X-2)
- kubectl can be of version (X+1 , X, X-1)
- Whever you are updating update one version at a time
	- Suppose you are on version v1.10 and v1.13 has been released
	- You need to upgrade to 1.10 to 1.11 then 1.11 to 1.12 and then to 1.13


### Security

- Authentication
- Authorization
- TLS
- Network Policies (By default all pods can talk to each other. To prevent that Network Policies can be set up)

#### Authentication
- Authentication  had 4 mechanisms (First 2 are depricated after k8s 1.19 )
  - Static password file  		(Passed as parameter to API server)
  - Static Token File			(Passed as parameter to API server)
  - Certificates
  - Service Accounts (Not in CKA)

##### Certificates
- Trust , Encrypt data (etc, etc)
- ssh-keygen 	--> 	for ssh keys
- open ssl 	--> 	for certificates

- Easy way to rememeber which is public and private key
  - Public Key -- `*.crt    *.pem`
  - Private Key -- `*.key    *-key.pem`
- Private keys has `key` in their name (please note this isnt compulsary but just a easy way if this naming convention is followed) 
- There are 2 types of certificates
  - **Server Certificate** 
  - **Client Certificate** 
- Server Certificates - `API etcd kubelet`
- Client Certificates - `Admin User scheduler controller kube-proxy and optional for (API and kubelet)`
- Server Certificates are to prove that they are they server. Client certificate is to prove they are right clients and authenticated to talk to right server.
- Health Check for certificates can be done there is some way please google
- Certificates are configured while running each component. things like server crts, client crts, trusted crts etc will be provided as parameters when you do this
- `/etc/kubernets/manifests/*` or `/etc/systemd/system/*` are the files to check

- To look into a crt file run the below command

~~~
openssl x509 -text -noout -in certificate.crt 
~~~ 

- Crt file has information like
  - Validity (Not Before and Not After)
  - Issuer (CA)
  - Name
  - Alternative Names (DNS, IPs)
 
- Sometimes when kubernets components are down beause of certificate error then you need to check `docker logs` for those to identify the issue

- There is certificate API run by k8s controlled by `kube-controller`
- Config file format to create `CertificateSigningRequest`
- kubectl commands to view, approve, deny, delete those

~~~
kubectl get csr
kubectl describe csr <name>
kubectl certificate approve <name>
kubectl certificate deny <name>
kubectl certificate delete <name>
~~~


#### Authorization

##### API Groups

~~~
 					Api  |   apis
 					     |
 					 API Groups		(/apps , /networking.k8s.io)
 					     |
 					 Resources		(/deployments,  /replicasets  (from/apps))
 					 	  |
 					 	Verbs
 	(create, delete, deletecollection, get, list, patch, update, watch)
~~~

#### Authorization Mechanisms in K8s

1. Node - For other nodes to node communication 
2. ABAC - Attribute based
3. RBAC - Role Bases
4. WebHook - Used by external tools like Open Policy Agent (OPA)
5. Always Allow
6. Always Deny

- To specify which authorization mode should be used you can input it in kube-api parameter `--authorization-mode= Node,RBAC,webhook`
- there can be multiple modes in comma seperated format. k8s will evaluate them sequencially and if the role is admitted it will break the chain and allow the operation else will continue the chain

##### RBAC

###### Roles and Role Binding

- Roles are namespaced
- If no namespaced is defined `default` namespace is used
- Kind `Role` Binding `RoleBinding `
- To check all the resoruces which falls under role run below command
`kubectl api-resources --namespaced=true`
- O/P - You will get apiGroups and Resources from here to be put into Role or ClusterRole Files

~~~
kubectl get roles
kubectl get rolebindings
kubectl describe role <Role Name>

## check access for yourself

kubectl auth can-i delete nodes

## check access for others

kubectl  auth can-i get pods --as dev-user -n default
~~~

###### Cluster Role and ClusterRoleBinding
- There are resources which are cluser wide hence Cluster role over Role
- Cluster Scoped resource example
	- Node
	- PV
	- CSR
	- Namespaces etc 
- Kind `ClusterRole` Binding `ClusterRoleBinding`
- To check all the resoruces which falls under ClusterRole run below command
`kubectl api-resources --namespaced=false`
- O/P - You will get apiGroups and Resources from here to be put into Role or ClusterRole Files

#### Image Security
- To pull image from a secure registry
	- Create a secret
	`kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
`
	- Under spec for pod `imagePullSecrets` specify the secret name


#### Security Context and PSP
- Why security context and pod security policy (PSP depricated 1.21 and will be removed in 1.25 in favour of Pod Security Standards -- PSS) 
	- [link] (https://medium.com/nerd-for-tech/getting-started-with-kubernetes-pod-security-context-and-policy-26619529a64b) 
- Security context is at pod level. Security context can also be at container level if this is set then this prevails over pod level
- PSP is cluster wide. It can control whether a pod can run on the cluster or not depending on the policies set up for the pod
- Security context
	- [link] (https://snyk.io/blog/10-kubernetes-security-context-settings-you-should-understand/) 

#### Network Policies
- By default all pods in Namespace can communicate with all pods across the Namespace
- **NOTE: Not all Network Solutions (CNI Plugins) support Netowrk Policies**
	- Supported
		- Kube-router
		- calico
		- Romana
		- Weave-net
	- Unsupported
		- As of now flannel doesnt supports Network Policies
	- Please check latest documentation when you are selecting it
	- Even if netowrk Sol. doesnt supports network policy you can still create it as it is k8s resource but it wont be enforced
- If you want to control this you can setup Network Policy
- Network Policy works at Namespace level
- One can think of Network policy as Security Group of any cloud
- It has Ingress and Egress rules which you can apply
- `Pod Selectors` is used to select which pods the network policy is applied to
- Example of Network Policy

~~~
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
~~~

- The important sections to look for
	- `name` and `namespace` in metadata
- `Spec`
	- `podSelector ` selects the pods to which the policy will apply
	- `policyTypes `  which kind of rules will this policy have
	- `ingress`
		-  `from`
			- `ipBlock`
				- If you have IP address outside the cluster which you want to communicate with
				- Example backup servers, external static apis etc
			- `podSelector `
				- Appropriate lables should be selected
			- `namespaceSelector `
				- By default if only pod selector is specified then it will cater to all the pods on the cluster (across all namespaces) having same lables but that wont be desrired
				- Hence you can tie down this behaviour with   namespaceSelector
				- If you only specify `namespaceSelector ` then all the pods in the same NS will be considered
				- Make sure Namespace has been labeled before using this
	- `egress`
		- Same as Ingress

- **Note** just as Json the structuring of `podSelector` and `namespaceSelector ` matters for **AND** and **OR** Rules



## Storage in Kubernetes

- Persistent Volume
- Persistent Volume Claims
- Storage Class (Not in scope of CKA)
- Stateful Sets (Not in scope of CKA)

**Note** For the scope of CKA only basic creation and attacting is in scope

### Docker Storage
- Storage Driver Plugin
- Volume Driver Plugin

#### Storage Driver

- Docker creates `/var/lib/docker` directory and stores its data inside that directory among various folders such as
	- containters
	- images
	- volumes
	- aufs

- Docker uses layred architecture for images so when you create a container it refers to the image layers and creates a container. These layers are in `Read Only` mode
- However if you exec inside a container and modify any files then these files are `Copy on Write` i.e a copy in `Read Write mode` is created and that is used which will be deleted when container is deleted
- Such kind of `Copy on Write` and layred architecture is executed by Stroage Driver
- Some of the storage drivers
	- aufs
	- zfs
	- btrfs
	- Device Mapper
	- Overlay
	- Overlay2
- Selection of Strage driver depends on the underlaying Operating System. Docker will automitically choose it.

#### Volume Driver
- Volume drivers help to manage the volume with docker
- Default is local which will create volume inside folder `/var/lib/docker/volumes`
- Other storage drivers
  - Azure File Storage, Convey, Flocker, NetApp, Portworx, GlusterFS, etc etc
  - RexRay -- can help to provision on Cloud like EBS in aws or Azure File Storage in Azure etc etc

#### Container Storage Interface (CSI)
- Similar to CNI (Network interface ) CSI is for storage
- Standarize way for container runtime and orchestration to work with each other so they can be mixed and matched


#### Persistent Volume
- Similar to docker volume can be mounted on host system in k8s. But this will mean that its attached to one node and not shared across nodes
- Also every team will create its own volume and manage which can become proble in bigger clusters where various teams work
- Hence persistent Volume usually think of this as Administrator creates a huge chunk of Volume and then team can chucks from it using Peristent Volume Claims
- Hence the storage option is centrally managed
- Persistent volume needs to be first created manually.
- Has different types of access modes
	- ReadOnlyMany
	- ReadWriteOnce
	- ReadWriteMany
- **Note** : The access mode also depends where you are provisioning the volume. For example like elastic Block Storage (EBS) doesnt supports mount on multiple Ec2 instance so you can only use `ReadWriteOnce`
- To check which `Volume Plugins` supports what scroll down in this [link] (https://kubernetes.io/docs/concepts/storage/_print/#access-modes)

#### Persistent Volume Claims (PVC)
- PVC and Persistet Volume (PV) are not directly binded to each other
- When PVC is created k8s looks for appropriate match in PV which it can allocate based on
	- Sufficient Capacity
	- Access Mode
	- Volume Mode
	- Storage Class
- So there can be cases when multiple PV might match to PVC then a random PV will be selected
- To avoid this one can use `labels and selectors`
- One PVC can be binded to only one PV
- If no PV is available then PVC will be in `pending state`
- `persistentVolumeReclaimPolicy`
	- What happens to the volume once PVC is deleted
	- `Retain` (Default) (Volume wont be available to anyone else unless administrator takes some action)
	- `Delete` (Volume Will be deleted)
	- `Recycle` (Not everyone supports this) (`rm -rf * ` will be performed and volume is available for others to use)
	- [Additional Details] (https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming)

## Cluster Installation

### ETCD
- Uses RAFT algorithm to select the leader
- Quorum is minimum nodes required to say write is successful
- Quorum = N/2 + 1  (1-1, 2-2, 3-2, 4-3, 5-3 etc)
- Fault Tolerance = No of Instances - Quorum  (1-0, 2-0, 3-1, 4-1, 5-2)
- Hence recommended to have ODD number of nodes (Network Segments)


## Troubleshooting
- Application Failure
- Control Plane Failures
- Worker node Failure
- Network Troubleshooting

### Application Failure
- Suppose if a pod is restarting so to view the logs fo previous pod

~~~
kubectl logs web -f --previous
~~~

**Note not directly into CKA exam but will be good handy trick**

- If pod is running and there are no debug utilities inside like not even `sh` or `/bin/bash` to login and debug then you can use ephemeral container
[link] (https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/#ephemeral-container)



-------------
#### Additional things to read
	- etcd
	  - backup and frequency
	  - security as it has secrets
	- Garbage collections 
	- Why Replica Sets were introduced over Replication Controller?
	  - Exsting pods with same lables can have different images
	  - isnt that an issue?
	- Best practies for labeling!
	- Encrypting data at rest (etc encryption)
	- How to give passwords in secret manner (Hashicorp vault?) read and understand more about it. k8s secrets arent safe
	- When node goes down how long does k8s waits for it?
	  - Default is 5 mins
	  - kube-controller-manager --pod-evection-timeout=5m0s 
	- PV and PVC if one creates a PV of 100mb and PVC of 50 then the PVC gets 100. will it be same if case of GB? how does that work?
	- k8s the hardway [link] (https://www.youtube.com/watch?v=uUupRagM7m0&list=PL2We04F3Y_41jYdadX55fdJplDvgNGENo&ab_channel=KodeKloud)
	  - Video 18 - End to end test case

----------------
### Helpful commands 

- To find k8s version

~~~
kubectl version --short

#>>> will give Client version and server version
#>>> Server version is the version of k8s
~~~

- To get help for k8s commands similar to `man` in linux

~~~
# kubectl explain <<resource_type>> --recursive

kubectl explain pod --recursive

#>>> kubectl explain pod --recursive | grep initContainers -A8
#>>> this is search for initContainers and display 8 lines below it.
#>>> Very useful when you want to see how to structure the defination file on fly 
~~~

- To get all the resoruces deployed

~~~
kubectl get all 

#>>> kubectl get all -n dev

#>>> Displays all the different resources deployed in the Namespace
#>>> (reverify) : I think this doest shows ingress, configMap, and secrets need to fire individual commands for them 
~~~

- To get config file from existing resource

~~~
kubectl get pod nginx -o yaml >> nginx_pod.yaml 
~~~

- To create a config file on fly

~~~
kubectl run nginx --image=nginx --dry-run=client -o yaml >> nginx_pod.yaml 
~~~

- Can change the default NS to point to something else by below command

~~~
kubectl config set-context $(kubectl config current-context) --namespace=<namespace>
~~~

- Check Access to kubernetes resoruces

~~~
## check access for yourself

kubectl auth can-i delete nodes

## check access for others

kubectl  auth can-i get pods --as dev-user -n default
~~~

- Check API Resources for Roles and Cluster Roles

~~~
kubectl api-resources --namespaced=true  # Role
kubectl api-resources --namespaced=false # ClusterRole

### O/P
You will get apiGroups and Resources from here to be put into Role or ClusterRole Files
~~~

- Suppose if a pod is restarting so to view the logs fo previous pod

~~~
kubectl logs web -f --previous
~~~

- If your pod is not behaving as you expected, it may be that there was an error in your pod description (e.g. mypod.yaml file on your local machine), and that the error was silently ignored when you created the pod. Often a section of the pod description is nested incorrectly, or a key name is typed incorrectly, and so the key is ignored. For example, if you misspelled command as commnd then the pod will be created but will not use the command line you intended it to use.

The first thing to do is to delete your pod and try creating it again with the --validate option. For example, run kubectl apply --validate -f mypod.yaml. If you misspelled command as commnd then will give an error like this

~~~
kubectl apply --validate -f mypod.yaml
~~~

- If service is not running properly then you can first check if it has desried number of endpoints (Example  if there are supposed to be 3 pods below the serive then you should see 3 endpoints)

~~~
kubectl get endpoints ${SERVICE_NAME}
~~~

- If suppose the above doesnt shows you desired endpoints then probably selectors with service is mismatched or misspelled you can check that
- If you are missing endpoints, try listing pods using the labels that Service uses. Imagine that you have a Service where the labels are:

~~~
...
spec:
  - selector:
     name: nginx
     type: frontend
~~~
**Note : No spaces allowed in the ,**

**Also even if endpoints are displayed ports can still be issue**

(Check no 3 qustion again)

~~~
kubectl get pods --selector=name=nginx,type=frontend
~~~

- Good way to start trouble shooting is looking at the troubleshooings steps in k8s documents

~~~
search keyworks like :
- Troubleshooting application
- Troubleshooting cluster
- Troubleshooting services
- etc
~~~