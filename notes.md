# Trello link
https://trello.com/b/kyi6vb5V/learn-kubernetes

# Kubernetes Components
- Clusters:
- Control Planes:
    - manage the cluster and the nodes that are used to host the running applications
    - schedules application instances from Deployments
    -contains
        * kube-scheduler
        * controller-manager
            - node-controller
            - replication-controller
        * etcd cluster
            - who did what when and why
        kube-apiserver
- Pod:
    - kubernetes abstraction that represents a group of one or more application containers
    - also represents shared resources
        * shared storage, such as Volumes
        * Networking (IP address for cluster)
        * image version, port specs, etc.
    - ephemeral and destroyed frequently
    - random ip from Node internal range
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
    - pods get serial names (ex: \<app\>-0, \<app\>-1)
    - pods get (individual dns name) endpoints
- Namespace
- ReplicaSet:
    - drives cluster to desired state through creation of Pods
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


# neato notes
- you can use the '-l' flag to kubectl get by label
    - ex: `$kubectl get services -l app=my-app`

# Common Commands (C'mon) from the Kubernetes Tuts
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

# Appendix:
- https://gitlab.com/nanuchi/youtube-tutorial-series/-/tree/master/
- https://www.youtube.com/watch?v=X48VuDVv0do
- https://kubernetes.io/docs/tutorials/
