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
 - Both are similar in primary functionality just that Replicat sets have additional field in configuration called `selector`
 - `selector ` is used to identify pods which will be run or already running by thier lables and then Replica sets starts to monitor them
 - catch here is pods need not be created by Replica set to be monitored but can be existing pods with the tags that `selector is looking for`
 - Replica set adds and checks for `ownerReferences`
   - `metadata.ownerReference`
   - Only if object doesnt have a owner i.e its orphaned then it takes it on the watch list.
   - `that tag` is also tied to garbage collection mechanism (TBD)
 - IDK why this was introduced as it can complicate things if existing pods are running different images! -- NEED TO READ MORE! 
 -

#### Namespaces 
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

Delarative is something that you just declare the state and let the engine decide how it wants to reach there. Or what kind of statements it wasnts to execute

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

Because all of these compliment each other and you need to understand the behaviour well to understan how it works.

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
      - **PreferNodSchedule** : Pods are prefered to not be scheduled but no gurantee
      - **NoExecute** : New pods wont be scheduled and existing pods will be deleted
    	  - NoExecute has additional field called `tolerationSeconds` 
    	  - If above field is set then k8s waits for that time before any pods are deleted 
- Think of Taint effect as what can happen if taint is applied before any pods are scheduled and what should happen if node had existing pods and then taint was applied! 

- K8s also uses taints as internal mechanism for kubernetes to work
  - Example master nodes are tainted to run only master components 
  - Whenever a node is unhealthy/unreachable or having any other issues/pressure like cpu,memory,io,pid . K8s internally sets taits to reflect the node behaviour
  - Based on this taints scheduler determines if pods are to be sent to these nodes or not! So in short unhealty status of node is reflected to scheduler via Taints! 

~~~
 kubectl describe node kubemaster | grep taint
 
 o/p should be something like>> node-role.kubernetes.io/master:NoSchedule
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

1. The affinity/anti-affinity language is more expressive. The language offers more matching rules besides exact matches created with a logical AND operation;
2. You can indicate that the rule is "soft"/"preference" rather than a hard requirement, so if the scheduler can't satisfy it, the pod will still be scheduled;
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
	- DoesNotExisits  
	- Gt
	- Lt

- Types of node Affinity
	- `requiredDuringSchedulingIgnoredDuringExecution`
	- `preferredDuringSchedulingIgnoredDuringExecution`
	- Future
		- `requiredDuringSchedulingRequiredDuringExecution ` 	
- What happens if there are no labels or lables are changed at certain later stage ?
  - Type of node Affinity determines this behaviour 



















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