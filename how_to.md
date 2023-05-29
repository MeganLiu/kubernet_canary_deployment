# kubernet canary deployment example

###
![diagram](https://github.com/MeganLiu/kubernet_canary_deployment.md/issues/1#issue-1730966067)

###
---
### first create an deployment named "myapp"
```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: myapp
  
spec:
  replicas: 3
  
  selector:
  
    matchLabels:
    
      app: myapp
      
  template:
  
    metadata:
    
      labels:
      
        app: myapp
        
    spec:
      containers:
      
      - name: myapp
      
        image: myapp:1.0
        
        ports:
        
        - containerPort: 8080
```
---
### second expose the deployment as servcie  
```
apiVersion: v1

kind: Service 

metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080

```
---
### create a new deployment  for  new image 'myapp:2.0" named "myapp-canary"
```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: myapp-canary
  
spec:


  replicas: 1
  
  selector:
  
    matchLabels:
    
      app: myapp-canary
      
  template:
  
    metadata:
    
      labels:
      
        app: myapp-canary
        
    spec:
    
      containers:
      
      - name: myapp
      
        image: myapp:2.0
        
        ports:
        
        - containerPort: 8080

---
### define Ingress service

apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: myapp-ingress
  
  annotations:
  
    nginx.ingress.kubernetes.io/canary: "true"
    
    nginx.ingress.kubernetes.io/canary-by-header: "Cookie"
    
    nginx.ingress.kubernetes.io/canary-by-header-value: "canary"
    
spec:

  rules:
  
  - http:
  - 
      paths:
      
      - path: /
      - pathType: Prefix
      - 
        backend:
        
          service:
          
            name: myapp-canary
            
            port:
            
              number: 80
```
