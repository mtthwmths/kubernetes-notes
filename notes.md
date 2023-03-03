# Trello link
https://trello.com/b/kyi6vb5V/learn-kubernetes

# Kubernetes Components
- Clusters:
- Control Planes:
    - manage the cluster and the nodes that are used to host the running applications
    - schedules application instances from Deployments
    -contains
        * kube-scheduler
            - runs as a pod in the kube-system namespace typically
        * controller-manager
            - node-controller
            - replication-controller
        * etcd cluster
            - who did what when and why
        kube-apiserver
- Pod:
    - remember the kubectl replace --force command will delete an older pod and create the one you are specifying in your new yaml file.
    - kubernetes abstraction that represents a group of one or more application containers
    - also represents shared resources
        * shared storage, such as Volumes
        * Networking (IP address for cluster)
        * image version, port specs, etc.
    - ephemeral and destroyed frequently
    - random ip from Node internal range
    - pods are placed on a Node by the scheduler
- Jobs:
    - a job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate
    - Suspending a job will delete the active pods until it is resumed
    - deleting a Job will clean up any pods it created
- Binding:
    - these can be used to assign a nodeName to a pod if there is no scheduler
    - apiVersion:v1, kind:Binding, metadata:{name:''},target:{apiVersion:v1, kind:Node, name:''}
- Node:
    - a VM or a physical computer that serves as a worker machine
    - each Node has a Kubelet and a container runtime (docker, containerd, rkt)
    - can have multiple Pods
    - each worker Node has a kube-proxy
- Kubernetes API (kube-apiserver):
    - Nodes communicate with the Control Plane using this
    - End users can also use this to interact with the cluster
    - this is what kubectl interacts with
- Kubelet:
    - listens for instructions from kube-apiserver
    - also reports the node status back to kube-apiserver
    - makes sure containers are running in a Pod for a Node.
    - manages the Node and communicates with the Control Plane
- kube-proxy
    - allows worker nodes to communicate with each other across the cluster
- Service: 
    - how to abstract away apps using multiple Pods.
    - let's talk about ports
        - targetPort is the port that is running on your app's container.
            - ex: nginx container running in a pod exposed on the pods port 80. you want to expose that port in a service, you'll need to set targetPort to 80.
        - port is what your service is exposing to the node
            - ex: an app in your same node wants to access a service 'nginx-service' where the port is set to 80. This app would be looking for nginx-service:80.
        - NodePort is the port that your node has exposed for outsiders wanting the service. 
            - ex: 'nginx-service' has nodePort set to 31245, and the node is running on a machine with ip of 10.10.0.20. outsider can get to nginx-service by going to 10.10.0.20:31245 :)
    - defines a logical set of pods and a policy by which to access them
    - defined using YAML (or JSON)
    - a set of Pods targeted by a Service is usually determined by a LabelSelector
    - load balancing
    - match a set of Pods using labels and selectors
        * Labels are key/value pairs attached to objects
            - designate dev/test/production
            - embed version tags
            - classify an object using tags
    - several types
        * ClusterIP Services
            - this is the default type
        * Headless Services
            - ex: stateful apps like databases
            - you can reach the pod without need of a proxy
        * NodePort Services
            - external traffic has access to fixed port on each Worker Node
            - not secure by nature
            - extends the ClusterIP Service
            - typically used to test connections, but not for production use
        * LoadBalancer Services
            - uses cloud providers LoadBalancer
            - NodePort and ClusterIP services are created automatically by Kubernetes
            - extends the NodePort Service        
        * ExternalName
            - maps the service to the contents of the 'externalName' field by returning a CNAME record with its value
            - no proxying of any kind is set up.
            - requires v1.7 or higher of kube-dns
            - typically used without a selector to allow for manually mapping endpoints
- Deployment: 
    - instructs Kubernetes how to create and update instances of your application
    - pods get random hashes
    - Deployment controllers replace instances and provide a self-healing mechanism (for failure or maintenance)
- Ingress:
    - handles external requests by passing to services
- PersistentVolumeClaim
    - storage requested by kubernetes for pods
    - should be in the same namespace as the pod
- PersistentVolume
    - mapping from persistenvolume and pvclaim is always one to one
    - even when claim is deleted, the PV remains and will not be reused by any claims
    - mounted to a storage class
- StatefulSet: 
    - pods get serial names (ex: <app>-0, <app>-1)
    - pods get (individual dns name) endpoints
- Namespace
- ETCD:
    - distributed key-value store
    - listens on 2379 by default
    - etcdctl client is the command line tool for etcd
    - stores info about the cluster
        * Nodes
        * PODs
        * Configs
        * Secrets
        * Accounts
        * Roles
        * Bindings 
        * Others
    - etcd.service configuration must be set for High Availability clustes where there will be more than one
    - can be on the controlPlane (stacked) or external (external (lol(hilarious)))
- Replication Controller
    - helps run multiple copies of a pod or automatically bring up a new pod if one fails
    - older technology that is being replaced by Replica Set
- ReplicaSet:
    - drives cluster to desired state through creation of Pods
    - can use selectors, and is very smug about it when around RepController because they can't do that.
- Daemon set:
    - puts one instance of your pod on each node.
    - used for monitoring or logging to be sure that each node has the same ability.
    - kube-proxy
    - weave-net
    - very similar yaml setup to a ReplicaSet, with the spec.template section containing your pod definition
    - kubectl get daemonsets is your friend.
    - from v1.12 on for kube, daemon sets use node affinity and default scheduler to get pods on nodes.

# Youtube Interview Questions Notes
- Tell us about components of k8s
    - kubectl: talks through kube apiServer to your cluster
    - apiServer
        - first point of contact for rest commands
        - manages you cluster
    - ControllerManager
        - responsible for regulating the cluster
    - Scheduler 
        - schedules tasks for your worker nodes
    - ETCD
        - key value storage of cluster state and spec
    - kubelet
        - controls the worker node
    - container runtime
        - on each worker node
        - makes containers
    - kube proxy
        - directs traffic on worker nodes
        - load balances
    - pod
        - the most basic unit of k8s
        - can have 1 or more component
- Secrets are Encoded and not Encrypted
- Sematext Docker agent is a log collection agent (outdated info: kubectl log gets logs from stout and logrotater keeps them fresh)
- How can I keep a 20s Job from taking 5 minutes
    - you can use --activeDeadlineSeconds flag in your container spec to limit job run time.

# Udemy Class Notes

## Imperative vs Declarative
- Imperative
    - specify what to do
    - kubectl run, create, expose, edit, scale, set, create, replace, delete
        - all of these are imperative commands. You are telling the system what changes to make one at a time.
        - these are helpful in the certification exam or for small one-time changes used when testing some change before it goes into your yaml or configuration files.
        - update the definition that kubernetes has for components in it's internal yaml files, but doesn't update YOUR yaml files.
- Declarative
    - just ask for the destination
    - kubectl apply is used declaratively
    - everything is handled in your yaml files, and apply looks at the existing configuration and figures out what needs to be changed.

## Labels and Selectors
- I'm going to use the replicationSet/replicaset-definition.yml file as an example
- the metadata:{labels:{app:myapp, type:front-end}} labels (lines 3-7) are used to label the replica set.
- the spec:{template:{metadata:{labels:{app:myapp, type:front-end}}}} labels (lines 12-14) are the labels in our pods
- the spec:{selector:{matchLabels:{type:front-end}}} label (line 22) is how the replicaset knows which pods to monitor.
- ex: if you already had a pod with the label type:front-end in it, the replicaset would only need 2 new pods to meet the replicas:3 requirement in our yaml file.
- note: annotations are used at the same level under metadata in your yaml, but they contain notes such as version numbers, contact emails, etc. and are only informational

## taints and tolerations
- the example given in the udemy course made it clear for me.
    - bugs want to land on people. 
    - The person sprays a repellant (taint). 
    - some bugs have no tolerance for the taint and can't land. 
    - other bugs will be tolerant of the taint and can land.
    - the bugs are pods looking for a node. 
    - the person is a node that is filtering which pods can be placed on it.

## node affinity
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity
- you can label a node with kubectl label
- you can edit a deployment with kubectl edit, and the affinity goes on the pod spec not the deployment spec.
- you can generate a template yaml with kubectl create deployment blablabla

## resource requirements
- 0.5 cpu and 256Mi memory is the assumed minimum resource for a pod
    - you can specify as low as 0.1 cpu (100m). using the 'm' notation, you can go as low as 1m
    - 1 cpu is 1 aws vCPU, GCP Core, Azure Core
- specifying memory. using the G moves in groups of 1000, but using Gi uses groups of 1024
- 1 cpu and 512Mi is the assumem limit of resources for a pod
- when a pod exceeds limits, the cpu gets throttled to avoid exceeding the limit.
    - if a pod tries to consume memory than is limited constantly, it is terminated.
- for a Pod to get these defaults, you must set the default values by creating a LimitRange in the same namespace
    - https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/
    - https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

### quick note on editing pods and deployments
- you cannot edit the specifications of an existing pod other than
    1. spec.containers[*].image
    1. spec.initContainers[*].image
    1. spec.activeDeadlineSeconds
    1. spec.tolerations
- to edit a pod, you can either use the kubectl edit command and then delete->redeploy from the temporary yaml file or you can use kubectl get to output the yaml and then use that when you delete->redeploy
- with deployments, you can edit any property of the pods, because the deployment will handle deleting any pods that are out of the updated spec and redeploy them for you.

## static pods
- you can configure the kubelet to read the pod definition files from a directory on the server designated to store information about pods (manifests)
- the kubelet periodically checks this directory, and can create the pods from the definitions it finds there.
- if you remove a definition from this directory, it is deleted automatically.
- the pods created this way are called static Pods
- the directory is an option called --pod-manifest-path and is set on kubelet.service
    - you could also use the --config option to specify a separate yaml file that contains a staticPodPath: <directory> line
    - typically found in /var/lib/kubelet/config.yaml
- you can use docker ps to view the static pods created this way.
- the API server uses an ACDP API endpoint to provide input to kubelet
- if the node is part of a cluster, the static pods seen in kubectl get pods are a read-only copy. 
    - you would need to edit the static pod definition as described above to edit it.
- static pods and daemonsets are ignored by the kube-scheduler.
- differences between the two
    - creation
        - static pods are created by the kubelet
        - daemonsets are created by kube-api server (DaemonSet controller)
    - common usage
        - kubelet deploy control plane components as static pods
        - daemonsets deploy monitoring/logging agents on nodes
- ownerreference section will have the kind as Node. That's usually a static pod.
    - replicaset, daemonset, bla will be not a static node.
- an easier way to identify is in the name of the pod, typically static pods will have the node name appended to the end of it.

## multiple schedulers
- kubernetes cluster can have multiple schedulers
- you can specify schedulers in pod definition
- this looks *REALLY HARD* on first watch... :(
    - read along with https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/
    - this should be more verbose and hopefully clear it up
    - also consider doing the lab and just researching as you go.
- you can view which scheduler scheduled a pod by using `$kubectl get events -o wide`
    - also `$kubectl logs my-custom-scheduler --name-space=kube-system`

## monitoring and logging
- kubernetes does not come with a full-featured built-in monitoring solution
- 3rd party options
    - metrics server (vs heapster(deprecated))
        - you can have 1 metrics server per cluster
        - slimmed down when compared to heapster
        - in-memory monitoring solution (not stored on disk. no history)
        - the kubelet contains cadvisor(container advisor)
        - metrics-server is an addon in minikube
        - kubectl top node/pod
    - prometheus
    - elastic stack
    - datadog
    - dynatrace
- kubectl logs command shows logs from container on a pod
    - if there are multiple containers on a pod, you'll need to specify by container name
    - use it like `$kubectl get logs -f podname` to stream the logs as they update.

## Application lifecycle management

### rollouts and rollbacks
- kubectl has a rollout command
- 2 types of deployment strategies
    - Recreate: bring down all of the instances; outage; updated version
    - rolling update: the assumed strategy (Default) bring down an instance; updated instance replaces the one that was brought down; repeat for all instances
- commands:
    - imperative: kubectl set image
    - declaritive: kubectl apply
- under the hood, deployments create a separate replicaset to perform the rolling update.
- if strategy:type: rollingupdate then set rollingupdatestrategy: X% max unavailable, X% max surge
    - this tells you how many can be down at a time during an upgrade or rollback.

### configure applications

#### commands and arguments
- refresh on commands, arguments, entrypoints in docker
- `$docker run ubuntu` runs and exits immediately
- `$docker ps` will show nothing
- containers are not meant to hold an operating system, and once the service within is done or stops, the container is destroyed
- the docker file uses bash as the default command of the ubuntu image, this means it launched the bash program but no terminal was found and it exited. Since the command finished, the container was destroyed.
- `$docker run ubuntu sleep 5` this will keep it around for 5 seconds because you have specified a new command other than bash.
- this can be put in the dockerfile
    - use the ENTRYPOINT ["sleep"] to tell the dockerfile what to run as the image is started.
    - now you can run `$docker run ubuntu 10` and it will sleep for 10
    - put the CMD ["5"] after the ENTRYPOINT line to have it sleep for 5 seconds if no number is provided.
    - you can also change the entrypoint with `$docker run --entrypoint` flag
- creating a pod with this modified image can be put in the spec:containers:args section of your yaml file
    - to override the entrypoint, you can use spec:containers:command in your yaml file
- you cannot change the args of a pod using `$kubectl edit`

#### configuring environment variables
- huge shoutout to the nana youtube video. This is something I've seen already and that's solely thanks to that video.
- instead of key: and value: in your env: section, you'll be using key: and valueFrom:
##### configmaps
- configmaps are used to pass environment values in as key:value pairs
- there are two phases in using these. creating the configmap and injecting them into the pod.
- kubectl has a `create configmap` command.
    - make a config map to set APP_COLOR to darkblue
        - `$kubectl create cm <cm-name> --from-literal="APP_COLOR=darkblue"`
    - configMap files have the usual apiVersion, kind, and metadata, but they also have the new data: section
        - the data section will have things like `app_color: blue` or `max_allowed_packets: 100`
- to inject into the pod, you can use the envFrom: section under your pod's spec: section
    - configMapRef:{name: app-config}
- the lab used the envFrom to get all the key:value pairs from a configMap into your pod
    - see https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables
##### secrets 
- secrets are used to store sensitive information
- 2 steps just like configmaps: create it, inject it
- same yaml structure as configmap, but secret key:value pairs will have the value encoded
    - you can convert data from plain to encoded with:
        - `$echo -n 'mathis' | base64`
- if used as volumes, each secret will be a file in the volume that contains the value
- secrets are not encrypted, only encoded
    - do not check-in secret objects to SCM
- secrets are not encrypted in ETCD
    - consider enabling encrypting secretdata at rest.
- anyone able to create pods/deployments in the same namespace can access the secrets.
    - consider enabling role-based access control
- also consider using third-party secrets store providers in AWS, Azure, GCP, etc
- there are better ways of handling sensitive date like passwords such as using tools like Helm Secrets, HashiCorp Vault, etc.
- definitely skim the k8s.io/docs on secrets here: https://kubernetes.io/docs/concepts/configuration/secret/#risks
- configmaps have configMapRef, bet you can't guess what secrets have... (secretRef)
- demo on encrypting secret data at rest
    - to install etcdctl `$apt install etcd-client`
    - you'll need ca.crt, server.crt, server.key to get the secret from etcd
    - to get the secret from etcd: `$ETCDCTL_API=3 etcdctl --cacert=<path-to-ca.crt> --cert=<path-to-server.crt> --key=<path-to-server.key> get /registry/secrets/<namespace>/<secret-name> | hexdump -C`
    - this shows the secret in etcd in plain-text
    - also, you can use kubectl and run: `$kubectl get secret <secret-name> -o yaml` to view it in plain text there.
    - the point of encrypting secret data at rest is to avoid these vulnerabilities. :)
    - using a kubeadm setup, you can search for kube-apiserver.yaml in /etc/kubernetes/manifests to see if the --encryption-provider-config is set.
        - or you can do a `$ps -aux | grep kube-api | grep "encryption-provider-config"` to look for it.
        - this would show if encryption provider is setup
    - to set this up, the EncryptionConfiguration yaml file has a resources: section where you can specify what gets encrypted.
        - this can be secrets or whatever you need to encrypt.
        - also in that file, if the first thing in the list of resources:{providers:{}} is `identity{}` then you are not encrypting.
        - the secret needed in setting up your encryption provider can be generated using 
            - `$head -c 32 /dev/urandom | base64`
    - read along here if needed: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

#### multi-container pods
- to create a multi-container pod, simply add a new container in the spec:containers: section of your pod's yaml.
- multi-container pods fall under three patterns
    1. sidecar
    1. adapter
    1. ambassador
- this is more of a CKAD topic and not discussed in length in my CKA udemy course.
##### initContainers
- runs a process to completion in a container without the surrounding pod closing
- example: a process that pulls code from a repository, or that waits for an external service to be up before starting the actual application
- specified inside an initContainers section within the pod's spec section

#### self healing applications
- Liveness and Readiness Probes are not required for the CKA exam and will instead be covered in the CKAD course.

## Cluster Maintenance

### Operating System Upgrade
- I am falling asleep at the keyboard. send good vibes lol
- the pod-eviction-timeout is set on the controller-manager and typically waits 5 minutes before considering a pod dead
- you can drain a node with `$kubectl drain <node-name>`
- this allows the node to be restarted.
- you would need to uncordon the node to mark it as schedulable again
- cordon can be used to mark a node as unschedulable.

#### cluster upgrades
- components can be at different versions
- the kube-apiserver should be at the highest level.
- controller-manager and kube-scheduler should be 1 lower than that.
- kubelet and kube-proxy can be 2 lower than that.
    - ex:
        1. api 1.10
        1. scheduler 1.9 or 1.10
        1. proxy 1.8, 1.9, or 1.10
- typically if you deployed with kubeadm, you can use the tool to plan or apply an upgrade
- commands in these sections are on the master node unless stated otherwise.

#### uprgade the Master Node
- `$apt upgrade -y kubeadm=1.12.0-00`
- `$kubeadm upgrade plan`
- `$kubeadm upgrade apply v1.12.0`
- `$apt-get upgrade -y kubelet=1.12.0-00`
- `$systemctl restart kubelet`

#### upgrade a worker node
- `$kubectl drain node-1`
- `$apt-get upgrade -y kubeadm=1.12.0-00 #on the node-1 that was drained`
- `$apt-get upgrade -y kubelet=1.12.0-00 #on the node-1 that was drained`
- `$kubeadm upgrade node config --kubelet-version v1.12.0 #on the node-1 that was drained`
- `$systemctl restart kubelet #on the node-1 that was drained`
- `$kubectl uncordon node-1`

#### Backup
- etcd --data-dir is where all of the data lives
- `$ETCDCTL_API=3 etcdtcl snapshot save snapshot.db` 
    - you can always use `$export ETCDCTL_API=3` to avoid needing to type it in the command
- to restore
    - service kube-apiserver stop
    - `$etcdctl snapshot restore <filename> --data-dir <new path>`
        - the path of the data-dir is set to somewhere other than the old value
    - the restore creates a new etcd
    - set the data-dir in the etcd.service file to reflect the restore location <new path> from the earlier command
    - then systemctl daemon-reload
    - service etcd restart
    - service kube-apiserver start
- with all of the etcd commands, you'll need certificates and keys specified.
    - with a TLS-enabled ETCD database, the following are mandatory
        1. --cacert #verify certs of TLS-enabled servers using this CA bundle
        1. --cert #identify secure client using this TLS Cert file
        1. --endpoints=[127.0.0.1:2379] # this is the default as ETCD is running on master node and exposed on localhost2379
        1. --key #identify secure client using this TLS key file
    - ex: `$etcdctl --cacert="/etc/kubernetes/pki/etcd/ca.crt" --cert="/etc/kubernetes/pki/etcd/server.crt" --endpoints=https://[127.0.0.1]:2379 --key="/etc/kubernetes/pki/etcd/server.key" snapshot save /opt/snapshot-pre-boot.db`
- a "Stacked etcd topology" means the etcd nodes are colocated with controlplane nodes
    - if a control plane node (apiserver, controller-manager, scheduler) has an etcd pod on it as well, that is "Stacked"
- an etcd runs on separate hosts, and communicates through the apiserver of control plane nodes.
    - this decoupling of the control plane node and etcd node allows for more redundancy and resiliency, but costs twice the nomber of hosts when compared with the stacked strategy.
- that lab 2 though... hoo boy...
    - stacked topology vs external: know it
    - etcdctl: know how to use it.
    - if you're ssh-ing to a separate etcd server, you'll be using etcdctl.
        - don't bother looking for kubectl, all you want to know about is etcd. no kube required.
    - kubectl config: learn it, then... know it.
        - this is how you swap from context to context while on the "student-node" in the lab.
        - context is something you set up with kubernetes. some yaml somewhere. and it allows you to call resources in other nodes.
        - k config use-context is the star of this lab lolol
        - backup of etcd happens on control-plane nodes. You are on student-node or whatever.
            - this means that to backup the etcd, you need to ssh to the ip address of the controlplane node...
            - `$k describe -n kube-system po etcd-whatever #get the certs and keys for etcd, you'll need them for etcdctl`
            - `$k get no -o wide #get the ip`
            - `$ssh <the-ip>`
            - `$export ETCDCTL_API=3 && etcdctl <certs and keys> snapshot save <directory for backup>`
            - `$scp <the-ip>:<directory for backup> <local directory(/opt)>`
- whew, that was a LOT. definitely do some studying in this one before doing the CKA exam...

## Security
- oh man, this is a weakness of mine. Right up there with networking...
    - this section is scary.
### Security Primitives
- hosts must secure access and have password auth disabled, ssh-key based auth
- kube-apiserver is at the root of all access to the cluster. 
    - the first line of defense
- whe can access the apiserver
    - defined by the auth mechanisms
    - username/pass, username/token, certs, LDAP, service accounts
- what can they do with the access
    - RBAC auth (role based access control)
    - ABAC
    - Node auth
    - webhook
- the rest of the control plane is secured with TLS. more on that in a bit.
- all pods can access all other pods in the cluster
    - this can be restricted with network policies
### Authentication
- admins, developers, end users, 3rd party apps/bots
- secure between the internal components and these with authentication
- security of end users will be handled by the applications that or on the nodes so let's focus on the rest.
- let's segregate these into humans and non-humans (or users and service accounts)
- kubectl can manage service accounts with `$kubectl create serviceaccount`
- all user access is handled through apiserver, whether user is using kubectl or curling the apiserver directly
- static password and token files
    - this can be stored in a csv file with 3 columns (pass, user, id)
    - this will go in the kube-apiserver.service (or the kube-apiserver.yaml in /etc/kubernetes/manifests/ if using kubeadm)
    - this can be passed using curl in a direct request with the -u flag (value will be user:pass)
    - this is not recommended. consider volume mount while providing the auth file in a kubeadm setup
    - this basic setup is deprecated in 1.19 and removed n later releases
    - there will also need to be a rolebinding setup for these users (from the file)
    - serviceAccounts are covered in CKAD and not CKA. It is not part of the CKA curriculum.
### Notes On TLS
- asymmetric encryption (public and private key)
- certificate authority/ root certificates
- server certificates
- client certificates
- private keys have the word 'key' in the name typically. (with certificates or public keys having crt or pem)

# neato notes
- you can use the '-l' flag to kubectl get by label
    - ex: `$kubectl get services -l app=my-app`
- need to find a static pod on some node somewhere. got you
    - you'll need to know what node the pod is on
        - `$kubectl get pods -A -o wide`
    - now ssh to the internal IP of that node (assuming you're on controlplane)
    - staticPodPath should be in /var/lib/kubelet blabla
    - go to the directory for staticPodPath and then delete the file that is generating the pod.
    - after a bit the kubelet will see that file is gone and delete the file.
        - you can restart kubelet if you're feeling froggy.
            - `$systemctl restart kubelet`

# Common Commands (C'mon) from the Kubernetes Tuts
- make one pod with an image
    - `$kubectl run <podname> --image=nginx`
        - consider using something like `--dry-run=client -o yaml` on the end to get an example yaml file if you ever need one.
- you made a pod and oopsy, you need to change something imperatively
    - `$kubectl replace --force -f nginx.yaml #the force flag will delete the existing pod and create one with the config of the yaml`
- put podname in a variable:
    - `$export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')`
- API of the pod (assumes $POD_NAME variable is set):
    - (in a seperate terminal) `$kubectl proxy`
    - `$curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/`
- List environment variables of the pod (assumes $POD_NAME variable is set):
    - `$kubectl exec $POD_NAME -- env`
- start a bash session in the pod (assumes $POD_NAME variable is set):
    - `$kubectl exec -ti $POD_NAME -- bash`
- set the NodePort of a service to a bash variable
    - `$export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')`
- create a NodePort Service with kubectl
    - `$kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080`
- curl from a minikube NodePort service (assumes $NODE_PORT variable is set)
    - `$curl $(minikube ip):$NODE_PORT`
- check status of rollout
    - `$kubectl rollout status deployments/\<deployment-name\>`
- get a yaml file for something that already exists
    - `$kubectl get pod <pod-name> -o yaml > <filename>.yaml`
- what apiVersion or whatever is needed for <thing> (replicaset, ingress, whatever)
    - `$kubectl explain <thing>`
- yaml is hard, I want to make a deployment in CLI without needing one
    - `$kubectl create deployment nginx --image=nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`
- scale a deployment? sure.
    - `$kubectl scale deployment nginx --replicas=3`
- create a service named redis-service of type clusterip to expose pod redis on port 6379
    - `$kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml #this automatically uses the pod's labels as selectors`
    - `$kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml #this assumes selectors as app=redis. you cannot pass in selectors as an option`
- create a service named nginx of type nodeport to expose pod nginx's port 80 on port 30080 on the nodes
    - `$kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml #this will automatically use the pod's labels as selectors, but you cannot specify the node port.`
    - `$kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml #this will not use the pods labels as selectors.`
    - both of the above commands have their own challenges. kubectl expose is recommended, because it is easier to add a node port to the generated definition file.
- make a config map to set APP_COLOR to darkblue
    - `$kubectl create cm <cm-name> --from-literal="APP_COLOR=darkblue"`
- you can convert data from plain to encoded with:
    - `$echo -n 'mathis' | base64`
    - `$echo -n 'base64stuff==' | base64 --decode` will put it back to normal

# Appendix:
- https://gitlab.com/nanuchi/youtube-tutorial-series/-/tree/master/
- https://www.youtube.com/watch?v=X48VuDVv0do
- https://kubernetes.io/docs/tutorials/
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
- https://kubernetes.io/docs/reference/kubectl/conventions/
- https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
