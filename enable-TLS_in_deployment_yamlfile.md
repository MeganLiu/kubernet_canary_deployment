
### How Does Ingress TLS/SSL Work?
Adding TLS to ingress is pretty simple. All you have to do is,

#### Create a Kubernetes secret with server.crt certificate and server.key private key file.
#### Add the TLS block to the ingress resource with the exact hostname used to generate cert that matches the TLS certificate.
#### SSL is handled by the ingress controller, not the ingress resource. Meaning, when you add TLS certificates to the ingress resource as a kubernetes secret, the ingress controller access it and makes it part of its configuration.

For example, in the Nginx controller, the SSL certificates are dynamically handled by the following block in nginx.conf

ssl_certificate_by_lua_block {
			certificate.call()
		}
