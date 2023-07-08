ge the location to a directory which has index.html will fix the issue. Here is the working configmap with root   /usr/share/nginx/html. However this could be manipulated as you want, but we need to make sure files exist in the directory.


apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    user nginx;
    worker_processes  1;
    events {
      worker_connections  10240;
    }
    http {
      server {
          listen       80;
          server_name  localhost;
          location / {
            root   /usr/share/nginx/html; #Change this line
            index  index.html index.htm;
        }
      }
    }
