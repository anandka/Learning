#CKS - Kubernetes.md
~~~ 
Please note this is not exhaustive list.
But just pointers that I want to remember so I might have used shortcuts or just jotted down points which I feel are important.

For any references please search keywords in kubernetes documentation or cross verify things on Google :)
~~~


# Network policies
- Go through possible scenerios (Network policy isnt very exhaustive)
	- Good link to think about all possible scenerios [link](https://www.youtube.com/watch?v=3gGpMmYeEO8)
	- He also has github page describing the scenerios 
		- [github link](https://github.com/ahmetb/kubernetes-network-policy-recipes) 	
	- When we deny all default traffic does it still allows traffic via IP? is it only blocked for NS to NS communication?
	- What happens if the pod is expose via Service?
	- Also blocks Kube-dns you need rules like to allow it to work 
	
	~~~
	- ports:
		- port: 53
		  protocol: "UDP"
		- port: 53
		  protocol: "TCP"	  	
	~~~
	
	- Overhead of Network policy is sub-1ms depends on implementation
		- Caching
		- Overlay or Iptables

		
# Restrict access to GUI elements
- Tesla 2028 hack


# Secrets
- secrets are not secure by default in k8s
- secrets can be mounted as File/Volume or passed as env variables
- Hacks
	- if someone has docker access to the machine
		-  they can simply run `docker inspect` command and see the `ENV` variables and see the env secrets
		-  they can also just copy the files from the docker container and view the mounted files
	- if someone has access to master
		- they can connect to ETCD via command line
		- To access etcd you can get the certs etc from api server and use the same to run any etcd commands
		- by default etcd isnt encrypted to user can just view the secrets

## Encrypt ETCD (at rest)

- Encryption is achieved via kube-apiserver by parameter `-encryption-provider-config`
- As Apiserver is the only component which talks to etcd in k8s it makes sense. And then its responsibilty of api-server to encrypt and decrypt it for pods
- `-encryption-provider-config` pointed to a config file of kind `EncryptionConfiguration `
- The folder in which file is being pointed needs to be mounted on the apiserver 

~~~
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - identity: {}
    - aesgcm:
        keys:
        - name: key1
          secret: c2VjcmV0IGlzIHNlY3VyZQ==
        - name: key2
          secret: dGhpcyBpcyBwYXNzd29yZA==
~~~

- Note order of providers is important as to encode this order is followed.
- `identity: {}` means no encyption
- why many providers? you can have different encryption mechanims / keys at different times so while decoding each one is followed sequencially
- When creating a secret using base64 there can be "\n" introduced. So take care of that. Use "-n" with echo to skip the trailing newline.
`echo -n "The$Tr0nGPswRd | base64`

- **Encrypting secrets with a locally managed key protects against an etcd compromise, but it fails to protect against a host compromise. Since the encryption keys are stored on the host in the EncryptionConfig YAML file, a skilled attacker can access that file and extract the encryption keys.**
- Use other providers like kms and hashicorp vault to give stronger security
- Rotating a decryption key [link](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#rotating-a-decryption-key)

## Open Policy Agent (OPA)
- Opensource project can be used in k8s
- Request flow
	- Authentication
	- Authorization
	- Admission Controller
		- Mutating admission webhook (Checked first) (Ex: Istio automatic injection happens from here)
		- Validating admission webhook (approves or rejects) (Ex: OPA gatekeeper)
- Uses Rego language 
- CRD implentation in k8s called OPA Gatekeeper which uses admission controller in k8s
- There are 2 CRDs created for OPA
	- A CRD for declaring the policies (aka “constraint templates”)
	- A CRD for declaring how and when to enforce those policies (aka “constraints”)
- It also introduced new audit functionality to allow administrators to see what resources are currently violating any given policy for those things admitted before the policy was enforced



- PSP to gateway policies [link](https://github.com/open-policy-agent/gatekeeper-library/tree/master/library/pod-security-policy)
- To throw a violation every condition has to be true

- can we create OPA rules tied to Namespaces? Currently all examples are cluster wide

- not viewed [link](https://www.youtube.com/watch?v=urvSPmlU69k)
- caylent read [link](https://caylent.com/leveraging-kubernetes-open-policy-agent)

## mTLS (mutual TLS)
- By default in k8s any pod can communicate with any pod and this traffic isnt encrypted
- To encrypt the traffic between pod-to-pod communication we can use mTLS
- needs client and server certificate for every pod and CA to verify the certificates
- Rotation of cert is also important as we dont want long live certs which can be misused by a hacker
- Service Mesh/ Proxy / SideCar 
	- Service mesh is used to make mTLS wich injects a side car container into pod and which takes care of certificates etc.
	- Service mesh uses init containers which then play with IPTable rules for the pod to make all the traffic go via the proxy/sidecar container
	- this container needs `NET_ADMIN` capabilities

- Demystifying Istio's Sidecar Injection Model [link](https://istio.io/latest/blog/2019/data-plane-setup/)


## Runtime Security

### Behavioural analytics at host and container level (s24)

#### Strace 

- one can use strace to get the system calls which are being executed by the process

~~~
strace ls -l
strace 
	-o filename
	-v verbose
	-f follow forks
	
	-cw (count and summarise) (Gives summary in table of calls vs no of calls)
	-p pid
	-P path	
~~~

#### /proc
- directory in linux which stores all the process related information
- directy is created by `pid` id
- has subdirictories like
	- fd (open files)
	- environ (contains env variables) 
- **using fd if etcd is not encrypted then you can actually see the data like values of secrets created by just having access to the filesystem at host level**
- one can run `pstree` to get the root process and all its forks to identify which directory to go to in `/proc`
- `environ` file under the process directory shows all the env variables in plain text

##### Falco (Created by sysdig) [Kubernetes threat detection engine]

- What does Falco do? 
	- Parsing the Linux system calls from the kernel at runtime
	- Asserting the stream against a powerful rules engine
	- Alerting when a rule is violated 
- What does Falco check for? 
	- Privilege escalation using privileged containers
	- Namespace changes using tools like setns
	- Read/Writes to well-known directories such as /etc, /usr/bin, /usr/sbin, etc
	- Creating symlinks
	- Ownership and Mode changes
	- Unexpected network connections or socket mutations
	- Spawned processes using execve
	- Executing shell binaries such as sh, bash, csh, zsh, etc
	- Executing SSH binaries such as ssh, scp, sftp, etc
	- Mutating Linux coreutils executables
	- Mutating login binaries
	- Mutating shadowutil or passwd executables such as shadowconfig, pwck,chpasswd, getpasswd, change, useradd, etc, and others.

- Sample of syslog what falco does (also gives container id)
	- `tail -f /var/log/syslog | grep falco`
	- "Notice a Shell was spawned in a container with an attached terminal"
	- "Error File below etc opened for writing"
	- "Error package managment process launched in container"
- Details [official link](https://falco.org/docs/)

**Falco** can look at the cluster as whole and send out events/alerts depending on if certain container is misbehaving based on the rules

- By default installed under `/etc/falco`
- Rules
	- `/etc/falco/falco_rules.yaml`
	- `/etc/falco/k8s_audit_rules.yaml`
- Rules have macro and contitions. Conditions can refer to macro for evaluation
- Has `source` ex : k8s_audit so will work only if k8s auditing is enabled 


### Immutability of containers at runtime (s25)

- Container and Pod Level enforcement
- Ensure pods and containers are immutable

- There is no direct method but some hacks

- at container level
	- Remove shell / bash
	- Make filesystem read only
	- Run as user and not root
- But at times we dont have control over the docker images
- At k8s level we can run a `startupProbe` to run commands such as delete the executables or make file system readonly
- we can use init containers to load information (read write mode) before the main containers start and keep it read only for main container (as storage is shared)
- We can use `securityContext` to make `readOnlyRootFilesystem` this was in PSP which will be removed hence look at OPA? 
- **Read more on PSP and securityContext**
	  



-----------------

# Misc Read
- Logs at `/var/log/pods/` ?
- Istio [link](https://istio.io/latest/blog/2019/data-plane-setup/)
	- webhooks namespaceSelector (istio-injection: enabled)
	- default policy (Configured in the ConfigMap istio-sidecar-injector)
	- per-pod override annotation (sidecar.istio.io/inject)

- Sections [15, 18,19, ]