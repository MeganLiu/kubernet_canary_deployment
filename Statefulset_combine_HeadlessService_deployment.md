####  StatefulSets + Headless service (deploy stateful service MongoDB/Postgres)
StatefulSets require  to create a headless service to control their network identities.
### Ingress  + nginx ingress controller +  CloudLoadBalancer +  kubernet (Stateless) service  to reach loadbalanceing
A stateless system doesn’t modify its environment or write any persistent data. their container instances are interchangeable.
Kubernetes can also be used for stateful systems, e.g a database, a backend that writes files to persistent volumes, or a service where one replica is elected the leader to gain control of its neighbors.
StatefulSets provide several advantages over the ReplicaSet and Deployment controllers used for stateless Pods:
Reliable replica identifiers. Each Pod in a StatefulSet is allocated a persistent identifier. The Pod will retain its identifier even if it’s replaced or rescheduled, ensuring the new Pod runs with the same characteristics.
Stable storage access. Pods in a StatefulSet are individually assigned their own Persistent Volume claims. The Pod’s volume will be reattached after it’s rescheduled, providing stable storage access after a rollout or scaling operation.

### Rolling updates in a guaranteed order. StatefulSets support automated rolling updates in the order that Pods were created. newer Pods only replaced once older ones have updated.
Consistent network identities. Pods in StatefulSets have reliable network identities. Their hostnames include their numerical replica identifier, allowing external applications to interact with the same replica after a Pod’s rescheduled.
Kubernetes automatically creates, replaces, and deletes Pods as you scale the StatefulSet, while preserving any previously assigned identities. Replicated databases are a good example of the scenarios that StatefulSets accommodate. One Pod acts as the primary database node, handling both read and write operations, while additional Pods are deployed as read-only replicas.
Although each Pod may run the same container image, each one needs special configuration to set whether it’s in primary or read-only mode. This means your Pods possess their own state:
postgres-0 – Primary node (read-write).  postgres-1 – Read-only replica.
postgres-2 – Read-only replica.
###. When you use a StatefulSet, Kubernetes terminates Pods in the opposite order to their creation. This ensures it’ll be postgres-2 that’s destroyed first
A headless service is a service but has its clusterIP set to None. StatefulSets require a headless service to control their network identities.
 
 ### a DNS record for each pod associated with the service. These DNS records can then be used to directly address each pod, NS record for each pod <pod-name>.<headless-service-name>.<namespace>.svc.cluster.local.
  
  
  ## an example of statefulset+ headless service deployment for DB
 ``` 
  apiVersion: v1
kind: Service
metadata:
 name: my-db-service-headless
spec:
 clusterIP: None
 selector:
   app: my-db
 ports:
   - protocol: TCP
     port: 3306
     targetPort: 3306
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
 name: my-db-statefulset
spec:
 serviceName: my-db-service-headless
 replicas: 3
 selector:
   matchLabels:
     app: my-db
 template:
   metadata:
     labels:
       app: my-db
   spec:
     containers:
     - name: my-container
       image: my-db-image
       env:
       - name: MYSQL_ROOT_PASSWORD
         value: my-password
 volumeClaimTemplates:
 - metadata:
     name: my-pvc
   spec:
     accessModes: [ "ReadWriteOnce" ]
     resources:
       requests:
         storage: 10Gi
  ```
