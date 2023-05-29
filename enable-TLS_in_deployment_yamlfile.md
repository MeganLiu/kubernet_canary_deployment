
### How Does Ingress TLS/SSL Work?
Adding TLS to ingress is pretty simple. All you have to do is,

#### Create a Kubernetes secret with server.crt certificate and server.key private key file.
#### Add the TLS block to the ingress resource with the exact hostname used to generate cert that matches the TLS certificate.
#### SSL is handled by the ingress controller, not the ingress resource. Meaning, when you add TLS certificates to the ingress resource as a kubernetes secret, the ingress controller access it and makes it part of its configuration.

For example, in the Nginx controller, the SSL certificates are dynamically handled by the following block in nginx.conf
```
ssl_certificate_by_lua_block {
			certificate.call()
		}
		
		https://devopscube.com/wp-content/uploads/2022/03/ingress-tls-1.png
		
```		
```
kubectl create secret tls hello-app-tls \
    --namespace dev \
    --key server.key \
    --cert server.crt
```
Following is the equivalent YAML file where you have to add the crt and key file contents.
```
apiVersion: v1
kind: Secret
metadata:
  name: hello-app-tls
  namespace: dev
type: kubernetes.io/tls
data:
  server.crt: |
       <crt contents here>
  server.key: |
       <private key contents here>
```
	       
###. Add TLS block to Ingress Object
######## The ingress resource with TLS has to be created in the same namespace where you have the application deployed. So we create the example ingress TLS resource in dev namespace.

Save the following YAML as ingress.yaml. 
	       
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-app-ingress
  namespace: dev
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - demo.mlopshub.com
    secretName: hello-app-tls
  rules:
  - host: "demo.mlopshub.com"
    http:
      paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: hello-service
              port:
                number: 80
 ```
	       
As you can see, I have added the TLS block with the hostname (demo.mlopshub.com) and tls secret we created in the previous step. I have created the self-signed TLS certificate with emo.mlopshub.com domain.
	       
	       
```
tls:
  - hosts:
    - demo.mlopshub.com
    secretName: hello-app-tls
	      ```
The host in the TLS block and rules block should match.

Validate Ingress TLS
You can validate the Ingress TLS using the curl command as well as the browser.

From the CLI, run the curl command as given below with your domain name.

curl https://demo.mlopshub.com -kv
In the output, under server certificate, you can validate the certificate details as shown below.

validate ingress TLS using curl


