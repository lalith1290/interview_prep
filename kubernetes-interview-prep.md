# Kubernetes Interview Questions and Answers

## 1. Difference between Kubernetes and Docker

**Docker** is a containerization platform that packages applications and their dependencies into containers, ensuring they run consistently across environments.

**Kubernetes** is a container orchestration platform that automates deploying, scaling, and managing containerized applications across a cluster of nodes.

Key differences:
- Docker focuses on containerizing applications, while Kubernetes orchestrates containerized applications
- Docker manages individual containers on a single host, while Kubernetes manages containerized applications across multiple hosts
- Kubernetes can use Docker as its container runtime, but also supports alternatives like containerd and CRI-O

## 2. What is a Replication Set?

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. It's used to:
- Maintain the desired number of pods
- Ensure pod availability
- Scale applications horizontally

ReplicaSets use a selector to identify which pods to manage and a template to create new pods when needed.

## 3. Kubernetes Structure

Kubernetes consists of:
- **Control Plane**: Master node components that manage the cluster
- **Worker Nodes**: Execute applications in pods
- **Pods**: Smallest deployable units containing one or more containers
- **Services**: Enable network access to pods
- **ConfigMaps/Secrets**: Configuration and sensitive data management
- **Volumes**: Storage management
- **Namespaces**: Virtual clusters for resource isolation

## 4. Networking in Kubernetes

Kubernetes networking follows key principles:
- Every pod gets its own unique IP address
- Pods on a node can communicate with all pods on all nodes without NAT
- Agents on a node can communicate with all pods on that node

Kubernetes uses:
- **Pod Network**: For pod-to-pod communication
- **Services**: For stable endpoints to access pods
- **Ingress**: For external access to services
- **Network Policies**: For controlling traffic flow between pods

## 5. Replica Set

A ReplicaSet maintains a stable set of replica pods running at any given time. It's defined with:
- **Selector**: Labels used to identify which pods to manage
- **Replicas**: Desired number of pods
- **Template**: Pod specification for creating new pods

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

## 6. How to Get a Pod?

To get information about pods:

```bash
# List all pods in the current namespace
kubectl get pods

# List pods with more details
kubectl get pods -o wide

# Get detailed information about a specific pod
kubectl describe pod <pod-name>

# Get pod information in YAML format
kubectl get pod <pod-name> -o yaml
```

## 7. How to Launch a Pod in Kubernetes?

You can launch a pod using:

1. **Imperative approach**:
```bash
kubectl run nginx --image=nginx
```

2. **Declarative approach** (preferred) by creating a YAML file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

Then apply it:
```bash
kubectl apply -f pod.yaml
```

## 8. What is Kubernetes Architecture?

Kubernetes architecture consists of:

**Control Plane (Master) Components**:
- **API Server**: Exposes the Kubernetes API
- **etcd**: Distributed key-value store for cluster data
- **Scheduler**: Assigns pods to nodes
- **Controller Manager**: Runs controller processes
- **Cloud Controller Manager**: Interacts with cloud providers

**Worker Node Components**:
- **Kubelet**: Ensures containers are running in a pod
- **Kube-proxy**: Maintains network rules for service access
- **Container Runtime**: Software for running containers (Docker, containerd)

## 9. Kubernetes Scaling Deployment

To scale deployments:

1. **Imperative approach**:
```bash
# Scale to 5 replicas
kubectl scale deployment nginx-deployment --replicas=5
```

2. **Declarative approach**:
Update the deployment YAML file and apply:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 5  # Update this value
  ...
```

3. **Horizontal Pod Autoscaler (HPA)**:
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

## 10. Deployment Strategies in Kubernetes

Common deployment strategies include:

1. **Rolling Update** (default): Gradually replaces old pods with new ones
   - Zero downtime
   - Configurable update rate

2. **Recreate**: Terminates all existing pods before creating new ones
   - Results in downtime
   - Useful when applications don't support multiple versions running simultaneously

3. **Blue/Green**: Deploy new version alongside old version, then switch traffic
   - Requires duplicate resources
   - Quick rollback capability

4. **Canary**: Deploy to a small subset of users before full rollout
   - Limited impact if issues arise
   - Gradual rollout to validate changes

## 11. What is Node Affinity and Pod Affinity?

**Node Affinity**: Controls which nodes pods can be scheduled on based on node labels.
- `requiredDuringSchedulingIgnoredDuringExecution`: Hard requirement
- `preferredDuringSchedulingIgnoredDuringExecution`: Soft preference

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/e2e-az-name
          operator: In
          values:
          - e2e-az1
          - e2e-az2
```

**Pod Affinity/Anti-Affinity**: Controls pod placement relative to other pods.
- Affinity: Schedule pods near each other
- Anti-Affinity: Keep pods separated

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - frontend
      topologyKey: topology.kubernetes.io/zone
```

## 12. What is the Default Kubernetes Network?

Kubernetes doesn't come with a default network implementation but requires a Container Network Interface (CNI) plugin. Common options include:

- **Calico**: For policy-rich networks
- **Flannel**: Simple overlay network
- **Cilium**: eBPF-based networking
- **Weave Net**: Multi-host networking

When installing Kubernetes with tools like kubeadm, you must install a CNI plugin.

## 13. Commands in Kubernetes

Essential Kubernetes commands:

```bash
# Cluster information
kubectl cluster-info
kubectl get nodes

# Working with resources
kubectl get [resource]  # pods, deployments, services, etc.
kubectl describe [resource] [name]
kubectl create -f [filename]
kubectl apply -f [filename]
kubectl delete [resource] [name]

# Debugging
kubectl logs [pod-name]
kubectl exec -it [pod-name] -- /bin/bash
kubectl port-forward [pod-name] [local-port]:[pod-port]

# Scaling
kubectl scale deployment [name] --replicas=[count]

# Configuration
kubectl config view
kubectl config use-context [context-name]
```

## 14. How to Kill Kubernetes Pods?

To terminate pods:

```bash
# Delete a specific pod
kubectl delete pod <pod-name>

# Delete all pods in a namespace
kubectl delete pods --all

# Delete pods using label selectors
kubectl delete pods -l app=nginx

# Force delete a pod without grace period
kubectl delete pod <pod-name> --grace-period=0 --force
```

Note: Pods managed by controllers (Deployments, ReplicaSets) will be automatically recreated after deletion.

## 15. How to Deploy a Microservice in Kubernetes?

To deploy a microservice:

1. **Create a Deployment YAML**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-microservice
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-microservice
  template:
    metadata:
      labels:
        app: my-microservice
    spec:
      containers:
      - name: my-microservice
        image: my-microservice:1.0
        ports:
        - containerPort: 8080
```

2. **Create a Service for access**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-microservice
spec:
  selector:
    app: my-microservice
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

3. **Optional: Create an Ingress for external access**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-microservice-ingress
spec:
  rules:
  - host: myservice.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-microservice
            port:
              number: 80
```

4. **Apply the configurations**:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

## 16. Kubernetes fmt Command

Kubernetes doesn't have a built-in `fmt` command. You might be referring to:

- `kubectl apply --dry-run=client -o yaml`: Generate a formatted resource definition
- `kubectl get [resource] -o yaml`: Output resource configuration in YAML format

For formatting YAML files before applying:
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

## 17. How to Handle Replicas and Crash Issues?

To handle replicas and crash issues:

1. **Use appropriate resource requests/limits** to ensure pods have adequate resources.

2. **Configure health checks**:
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
```

3. **Set appropriate restart policies**:
```yaml
spec:
  restartPolicy: Always
```

4. **Use Pod Disruption Budgets** to ensure availability during voluntary disruptions:
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

5. **Check logs for crash debugging**:
```bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```

## 18. StatefulSet with PV

A StatefulSet with Persistent Volumes:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

This creates:
- Stable, unique network identifiers
- Persistent storage for each pod
- Ordered, graceful deployment and scaling

## 19. What is Ingress? Which Ingress Are You Working On?

**Ingress** is a Kubernetes resource that manages external access to services within a cluster, providing:
- Load balancing
- SSL termination
- Name-based virtual hosting

Popular Ingress controllers include:
- **Nginx Ingress Controller**: High-performance, feature-rich
- **Traefik**: Modern, cloud-native edge router
- **HAProxy**: Reliable, high-performance load balancer
- **AWS ALB Ingress Controller**: For AWS environments

Example Ingress resource:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: hello-world.info
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 8080
```

When interviewed, specify which Ingress controller you've worked with and your experience with it.

## 20. What is Kubernetes, and Why Use It?

**Kubernetes** is an open-source container orchestration platform that automates deploying, scaling, and managing containerized applications.

**Key benefits**:
- **Automated rollouts and rollbacks**: Safely deploy and update applications
- **Self-healing**: Restarts containers that fail, replaces and reschedules pods when nodes die
- **Horizontal scaling**: Scale applications manually or automatically
- **Service discovery and load balancing**: No need for custom service discovery mechanisms
- **Automated bin packing**: Efficiently utilizes resources
- **Secret and configuration management**: Deploy and update secrets and configuration without rebuilding images
- **Storage orchestration**: Automatically mount storage systems
- **Batch execution**: Manage batch and CI workloads

## 21. Kubernetes Architecture (Control Plane, Worker, and Master Nodes)

**Control Plane/Master Components**:
- **API Server**: Front-end for the Kubernetes control plane, exposing the API
- **etcd**: Consistent and highly-available key-value store for cluster data
- **Scheduler**: Watches for new pods and assigns them to nodes
- **Controller Manager**: Runs controller processes (node controller, replication controller, etc.)
- **Cloud Controller Manager**: Interfaces with cloud providers

**Worker Node Components**:
- **Kubelet**: Ensures containers are running in a pod
- **Kube-proxy**: Maintains network rules for pod communication
- **Container Runtime**: Software for running containers (Docker, containerd, CRI-O)

**Communication Flow**:
1. User interacts with API Server
2. API Server validates requests and persists state to etcd
3. Controllers watch for changes and maintain desired state
4. Scheduler assigns work to nodes
5. Kubelet on worker nodes creates and manages containers

## 22. Different Kubernetes Strategies

Kubernetes supports various strategies for different aspects:

**Deployment Strategies**:
- **RollingUpdate**: Gradually replace old pods with new ones
- **Recreate**: Terminate all existing pods before creating new ones
- **Blue/Green**: Run two versions side by side, then switch traffic
- **Canary**: Release to a subset of users before full deployment

**Scaling Strategies**:
- **Manual Scaling**: Using kubectl or updating manifests
- **Horizontal Pod Autoscaler (HPA)**: Scale based on CPU/memory usage
- **Vertical Pod Autoscaler (VPA)**: Adjust resource requests/limits
- **Cluster Autoscaler**: Add/remove nodes based on resource needs

**Pod Placement Strategies**:
- **Node Selectors**: Simple node assignment based on labels
- **Node/Pod Affinity/Anti-Affinity**: Complex rules for pod placement
- **Taints and Tolerations**: Repel pods from specific nodes unless tolerated

## 23. Kubernetes StatefulSet and DaemonSet

**StatefulSet**:
- Manages stateful applications with persistent identities
- Provides stable, unique network identifiers
- Ensures stable persistent storage
- Offers ordered, graceful deployment and scaling
- Suitable for databases, distributed systems

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

**DaemonSet**:
- Ensures all (or some) nodes run a copy of a pod
- Used for node-level operations
- Examples: log collectors, monitoring agents, storage plugins
- Pods are automatically added to new nodes

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
```

## 24. Deployment vs. StatefulSet

|Feature|Deployment|StatefulSet|
|-------|----------|-----------|
|Naming|Pods have random names (app-7d6f8d9f8-abcd)|Pods have predictable names (app-0, app-1)|
|Scaling|All pods are identical and interchangeable|Each pod has unique identity and state|
|Ordering|Parallel creation/deletion|Sequential creation/deletion|
|Storage|Shared storage or no persistence|Dedicated storage per pod|
|Network|Service load balances to any pod|Headless service for stable DNS names|
|Updates|Default strategy: RollingUpdate|Default strategy: RollingUpdate (ordered)|
|Use Case|Stateless applications|Stateful applications (databases, distributed systems)|

## 25. Commands for Kubernetes (Create Deployment, Pods, Services)

**Create Deployment**:
```bash
# Imperative
kubectl create deployment nginx --image=nginx --replicas=3

# Declarative
kubectl apply -f deployment.yaml
```

**Create Pod**:
```bash
# Imperative
kubectl run nginx --image=nginx

# Declarative
kubectl apply -f pod.yaml
```

**Create Service**:
```bash
# Expose a deployment as a service
kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP

# Declarative
kubectl apply -f service.yaml
```

**Other useful commands**:
```bash
# Get resources
kubectl get pods/deployments/services

# Delete resources
kubectl delete deployment/pod/service <name>

# Edit resources
kubectl edit deployment <name>

# Scale deployment
kubectl scale deployment <name> --replicas=5
```

## 26. Service Types in Kubernetes

Kubernetes offers four service types:

1. **ClusterIP (default)**:
   - Internal-only service, accessible within the cluster
   - Provides stable internal IP address
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     selector:
       app: MyApp
     ports:
     - port: 80
       targetPort: 9376
     type: ClusterIP
   ```

2. **NodePort**:
   - Exposes the service on each node's IP at a static port
   - Accessible from outside the cluster using <NodeIP>:<NodePort>
   - Port range: 30000-32767
   ```yaml
   spec:
     type: NodePort
     ports:
     - port: 80
       targetPort: 9376
       nodePort: 30007  # Optional
   ```

3. **LoadBalancer**:
   - Exposes service externally using cloud provider's load balancer
   - Automatically creates ClusterIP and NodePort services as needed
   ```yaml
   spec:
     type: LoadBalancer
     ports:
     - port: 80
       targetPort: 9376
   ```

4. **ExternalName**:
   - Maps service to external DNS name (no proxying)
   ```yaml
   spec:
     type: ExternalName
     externalName: my.database.example.com
   ```

## 27. How to Deploy MySQL DB in a Kubernetes Cluster

To deploy MySQL in Kubernetes:

1. **Create a Secret for MySQL password**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  password: cGFzc3dvcmQ=  # Base64-encoded "password"
```

2. **Create a PersistentVolumeClaim**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

3. **Create a Deployment or StatefulSet** (StatefulSet preferred for databases):
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 10Gi
```

4. **Create a Service**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mysql
  clusterIP: None  # Headless service for StatefulSet
```

5. **Apply all resources**:
```bash
kubectl apply -f mysql-secret.yaml
kubectl apply -f mysql-pvc.yaml
kubectl apply -f mysql-statefulset.yaml
kubectl apply -f mysql-service.yaml
```

## 28. PV and PVC in Kubernetes

**Persistent Volume (PV)**: A cluster resource representing storage in the cluster.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data/pv0001
```

**Persistent Volume Claim (PVC)**: A request for storage by a user.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

**Key characteristics**:
- PVs are cluster resources, PVCs are namespace resources
- PVs have lifecycle independent of pods
- PVs can be provisioned statically or dynamically
- StorageClass enables dynamic provisioning
- Different access modes: ReadWriteOnce, ReadOnlyMany, ReadWriteMany
- Different reclaim policies: Retain, Delete, Recycle

## 29. Kubernetes Manifest Files

Kubernetes manifest files are YAML or JSON documents that describe the desired state of Kubernetes objects:

**Key components**:
- **apiVersion**: API version of the resource
- **kind**: Type of resource (Pod, Deployment, Service, etc.)
- **metadata**: Resource identification (name, labels, annotations)
- **spec**: Desired state of the resource

**Example manifest structures**:

**Pod**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

**Deployment**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

**Service**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

## 30. How to Resolve Pod Issues

When troubleshooting pod issues:

1. **Check pod status**:
```bash
kubectl get pods
kubectl describe pod <pod-name>
```

2. **Check pod logs**:
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container pods
kubectl logs <pod-name> --previous  # Logs from previous instance
```

3. **Check events**:
```bash
kubectl get events --sort-by='.lastTimestamp'
```

4. **Debug with temporary pods**:
```bash
kubectl run debug --image=busybox --rm -it -- sh
```

5. **Common issues and solutions**:
   - **CrashLoopBackOff**: Check logs for application errors
   - **ImagePullBackOff**: Verify image name and credentials
   - **Pending status**: Check node resources and PVC binding
   - **Error/failed status**: Check events and describe pod
   - **Container not ready**: Check readiness probe and application health

6. **Execute commands inside containers**:
```bash
kubectl exec -it <pod-name> -- /bin/bash
```

## 31. How to Connect Two Kubernetes Nodes

Kubernetes nodes are automatically connected through the cluster network. To add a new node to the cluster:

1. **On control plane node**, generate a join token:
```bash
kubeadm token create --print-join-command
```

2. **On new worker node**, run the join command:
```bash
kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

3. **Verify connection**:
```bash
kubectl get nodes
```

For node communication troubleshooting:
- Ensure network plugin is properly configured
- Check firewall rules allow required ports
- Verify nodes can reach each other over the network
- Check kubelet status on both nodes

## 32. Explain Kubernetes Architecture

Kubernetes architecture consists of:

**Control Plane Components** (Master Node):
- **API Server**: Primary communication hub and validation
- **etcd**: Distributed key-value store for cluster data
- **Scheduler**: Assigns pods to nodes based on resource availability
- **Controller Manager**: Maintains cluster state through controllers
  - Node Controller: Monitors node health
  - Replication Controller: Maintains pod count
  - Endpoint Controller: Joins services and pods
  - Service Account & Token Controllers: Create accounts and API tokens

**Worker Node Components**:
- **Kubelet**: Node agent that ensures containers run in pods
- **Kube-proxy**: Network proxy and load balancer
- **Container Runtime**: Runs containers (Docker, containerd)

**Add-ons**:
- DNS for service discovery
- Dashboard for UI
- Networking solutions
- Monitoring and logging

**Workflow**:
1. User submits request to API Server
2. API Server authenticates, validates, and stores in etcd
3. Controllers detect changes and take necessary actions
4. Scheduler assigns work to appropriate nodes
5. Kubelet on worker node creates and manages containers

## 33. How Do You Access the Pod Through Browser?

To access a pod from a browser:

1. **Create a Service** to expose the pod:
   - **ClusterIP**: Internal access only (not accessible from browser outside cluster)
   - **NodePort**: Access via `http://<node-ip>:<node-port>`
   - **LoadBalancer**: Access via load balancer IP/hostname

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
  type: NodePort  # or LoadBalancer
```

2. **Create an Ingress** for path-based or host-based routing:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

3. **Port-forward** for temporary testing (not for production):

```bash
kubectl port-forward pod/<pod-name> 8080:80
```
Then access via `http://localhost:8080`

## 34. How Do You Check Logs of Pod?

To check pod logs:

```bash
# Basic logs
kubectl logs <pod-name>

# For specific container in multi-container pod
kubectl logs <pod-name> -c <container-name>

# Follow logs in real time
kubectl logs -f <pod-name>

# Show timestamps
kubectl logs --timestamps=true <pod-name>

# Show recent logs
kubectl logs --tail=100 <pod-name>

# Show logs from previous container instance (if crashed)
kubectl logs --previous <pod-name>

# Log filtering with grep
kubectl logs <pod-name> | grep ERROR
```

For persistent and cluster-wide logging, implement a logging solution like:
- Elasticsearch, Fluentd, and Kibana (EFK) stack
- Elasticsearch, Logstash, and Kibana (ELK) stack
- Grafana Loki
- Datadog or similar SaaS solutions
