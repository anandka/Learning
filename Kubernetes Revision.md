# Kubernetes Revision
~~~ 
Please note this is not exhaustive list.
But just pointers that I want to remember so I might have used shortcuts or just jotted down points which I feel are important.
For any references please cross verify things on Google )
~~~

Udemy Course - Mumshad Mannambeth

~~~
-------  3 Certifications -------
1. CKA - Administrator
2. CKAD - Application Developer
3. CKS - Security
~~~

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
  - Responsible to indetify nodes on which pods should be scheduled based 
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
   - Service is a virtual Component (No Pods run for it just a logical entity)
   
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

## Kubernetes Defination file 
At highlevel all the kubernetes defination file has 4 required components 

~~~ 
 apiVersion :
 kind: 
 metadata :
 spec:
~~~

All the information on above tag depends on what kind of resource you are going to make. So check the documentation as required.


#### Replica Sets
 - k8s previously used to have Replication Controllers 
 - As name suggests used to keep eyes on the pods and restart if any pods are killed or dies. To bring pods to desired state
 - Replica Sets should be used over Replication Controller
 - Both are similar in primary functionality just that Replicat sets have additional field in configuration called `selector`
 - `selector ` is used to identify pods which will be run or already running by thier lables and then Replica sets starts to monitor them
 - catch here is pods need not be created by Replica set to be monitored but can be existing pods with the tags that `selector is looking for`
 - IDK why this was introduced as it can complicate things if existing pods are running different images! -- NEED TO READ MORE! 

#### Namespaces 
 - to keep things inside same bucket! 
 - Namespaces share the networking and Storage
 - quota requirment can be implemented on top of Namespace
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
3 type of services
1. Node Port
2. Cluster IP
3. LoadBalancer
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