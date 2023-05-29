# kubernet canary deployment example

###
![diagram](https://github.com/MeganLiu/kubernet_canary_deployment.md/issues/1#issue-1730966067)

###
---
### first create an deployment named "sample-deployment"
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment
  labels:
    app: nginx
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
        - containerPort: 8080
```
---
### second expose the deployment as servcie  
```
apiVersion: v1
kind: Service
metadata:
  name: sample-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

```
### define Ingress service to route traffice to 'nginx' deployment
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-example
  annotations:
    kubernetes.io/ingress.class: nginx
  labels:
    app: nginx

spec:
  rules:
  -host: example.com
  - http:
      paths:
      - path: /
        backend:
          service:
            name: sample-service
            port:
              number: 80
```
---
### create a new deployment  for  new image named "canary-deployment"
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: canary-deployment
  labels:
    app: nginx-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-canary
  template:
    metadata:
      labels:
        app: nginx-canary
    spec:
      containers:
      - name: nginx-canary
        image: nginx:1.23.1
        ports:
        - containerPort: 8080

```
### expose canary deployment as canary service
```
---
apiVersion: v1
kind: Service
metadata:
  name: canary-service
spec:
  selector:
    app: nginx-canary
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
### define Ingress service for canary with "nginx Ingress controller", set weight to be 20

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: canary-ingress
  label:
    app: nginx-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  rules:
  -host: example.com
  - http:
      paths:
      - path: /
        backend:
          service:
            name: canary-service
            port:
              number: 80
```
