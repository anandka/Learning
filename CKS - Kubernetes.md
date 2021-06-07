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
	-
	-
	- 	
- Uses Rego language 
- CRD implentation in k8s called OPA Gatekeeper which uses admission controller in k8s


Constraint Template
crd - 

- To throw a violation every condition has to be true

- can we create OPA rules tied to Namespaces? Currently all examples are cluster wide



## mTLS (mutual TLS)
- By default in k8s any pod can communicate with any pod and this traffic isnt encrypted
- To encrypt the traffic between pod-to-pod communication we can use mTLS
- needs client and server certificate for every pod and CA to verify the certificates
- Rotation of cert is also important as we dont want long live certs which can be misused by a hacker
- Service Mesh/ Proxy / SideCar 
	- Service mesh is used to make mTLS wich injects a side car container into pod and which takes care of certificates etc.
	- Service mesh uses init containers which then play with IPTable rules for the pod to make all the traffic go via the proxy/sidecar container
	- this container needs `NET_ADMIN` capabilities




-----------------
Misc
- Logs at `/var/log/pods/` ?		