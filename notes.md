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
            - maps the service to the contents of the 'externalName' field by returning a CNAE record with its value
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
- PersistentVolume
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
- Replication Controller
    - helps run multiple copies of a pod or automatically bring up a new pod if one fails
    - older technology that is being replaced by Replica Set
- ReplicaSet:
    - drives cluster to desired state through creation of Pods
- Daemon set:
    - puts one instance of your pod on each node.
    - used for monitoring or logging to be sure that each node has the same ability.
    - kube-proxy
    - weave-net
    - very similar yaml setup to a ReplicaSet, with the spec.template section containing your pod definition
    - kubectl get daemonsets is your friend.
    - from v1.12 on for kube, daemon sets use node affinity and default scheduler to get pods on nodes.

# Imperative vs Declarative
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

# Labels and Selectors
- I'm going to use the replicationSet/replicaset-definition.yml file as an example
- the metadata:{labels:{app:myapp, type:front-end}} labels (lines 3-7) are used to label the replica set.
- the spec:{template:{metadata:{labels:{app:myapp, type:front-end}}}} labels (lines 12-14) are the labels in our pods
- the spec:{selector:{matchLabels:{type:front-end}}} label (line 22) is how the replicaset knows which pods to monitor.
- ex: if you already had a pod with the label type:front-end in it, the replicaset would only need 2 new pods to meet the replicas:3 requirement in our yaml file.
- note: annotations are used at the same level under metadata in your yaml, but they contain notes such as version numbers, contact emails, etc. and are only informational

# taints and tolerations
- the example given in the udemy course made it clear for me.
    - bugs want to land on people. 
    - The person sprays a repellant (taint). 
    - some bugs have no tolerance for the taint and can't land. 
    - other bugs will be tolerant of the taint and can land.
    - the bugs are pods looking for a node. 
    - the person is a node that is filtering which pods can be placed on it.

# node affinity
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity
- you can label a node with kubectl label
- you can edit a deployment with kubectl edit, and the affinity goes on the pod spec not the deployment spec.
- you can generate a template yaml with kubectl create deployment blablabla

# resource requirements
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

## quick note on editing pods and deployments
- you cannot edit the specifications of an existing pod other than
    1. spec.containers[*].image
    1. spec.initContainers[*].image
    1. spec.activeDeadlineSeconds
    1. spec.tolerations
- to edit a pod, you can either use the kubectl edit command and then delete->redeploy from the temporary yaml file or you can use kubectl get to output the yaml and then use that when you delete->redeploy
- with deployments, you can edit any property of the pods, because the deployment will handle deleting any pods that are out of the updated spec and redeploy them for you.

# static pods
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

# multiple schedulers
- kubernetes cluster can have multiple schedulers
- you can specify schedulers in pod definition
- this looks *REALLY HARD* on first watch... :(
    - read along with https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/
    - this should be more verbose and hopefully clear it up
    - also consider doing the lab and just researching as you go.
- you can view which scheduler scheduled a pod by using `$kubectl get events -o wide`
    - also `$kubectl logs my-custom-scheduler --name-space=kube-system`

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

# Appendix:
- https://gitlab.com/nanuchi/youtube-tutorial-series/-/tree/master/
- https://www.youtube.com/watch?v=X48VuDVv0do
- https://kubernetes.io/docs/tutorials/
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
- https://kubernetes.io/docs/reference/kubectl/conventions/
- https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource
