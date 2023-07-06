Helm chart is  a collection of files that will have the description of Kubernetes clusters and resources.
Helm charts are a combination of Kubernetes YAML manifest templates and helm-specific files. 
You can call it a helm package. 
Since the Kubernetes YAML manifest can be templated, you don’t have to maintain multiple helm charts of different environments. 
Helm uses the go templating engine for the templating functionality.

m Charts reduce the complexity and kubernetes manifest redundancy of 
each environment (dev, uat, cug, prod) with only one template.

Nginx-chart/
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- deployment.yaml
|   |-- configmap.yaml
|   |-- ingress.yaml
|   |-- service.yaml
|   `-- tests
|       `-- test-connection.yaml
`-- values.yaml

here are helm-specific files and folders. Let’s look at each file and directory inside a helm chart and understand its importance.

.helmignore: It is used to define all the files that not included in the helm chart. It works similarly to the .gitignore file.
Chart.yaml: It contains information about the helm chart like version, name, description, etc.
values.yaml: This file defines the values for the YAML templates. For example, image name, replica count, HPA values, etc. As we explained earlier only the values.yaml file changes in each environment. Also, you can override these values dynamically or at the time of installing the chart using --values or --set command.
charts: user can n add another chart’s structure inside this directory if the  main charts have some dependency on others. By default this directory is empty.
templates: This directory contains all the Kubernetes manifest files that form an application. These manifest files can be templated to access values from values.yaml file. Helm creates some default templates for Kubernetes objects like deployment.yaml, service.yaml etc, which we can use directly, modify, or override with our files.
templates/NOTES.txt: This is a plaintext file that gets printed out after the chart is successfully deployed. 
templates/_helpers.tpl: That file contains several methods and sub-template. These files are not rendered to Kubernetes object definitions but are available everywhere within other chart templates for use. 
templates/tests/: We can define tests in our charts to validate that your chart works as expected when it is installed. 

$helm create nginx-chart

Replace the default contents of chart.yaml with the following.

apiVersion: v2
name: nginx-chart
description: My First Helm Chart
type: application
version: 0.1.0
appVersion: "1.0.0"
maintainers:
- email: contact@xx.com
  name: xxx

apiVersion: This denotes the chart API version. v2 is for Helm 3 and v1 is for previous versions.
name: Denotes the name of the chart.
description: Denotes the description of the helm chart.
Type: The chart type can be either ‘application’ or ‘library’. Application charts are what you deploy on Kubernetes. Library charts are re-usable charts that can be used with other charts. A similar concept of libraries in programming.
Version: This denotes the chart version. 
appVersion: This denotes the version number of our application (Nginx). 
maintainers: Information about the owner of the chart.
You should increment the version and appVersion each time we make changes to the application.
There are some other fields also like dependencies, icons, etc. 

Remove all default files from the template directory.

rm -rf templates/*

add our Nginx YAML files and change them to the template for better understanding.
Create a deployment.yaml file and copy the following contents.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
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
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
in the above YAML file, the values are static. 
The idea of a helm chart is to template the YAML files so that we can reuse them in multiple environments by dynamically assigning values to them.

To template a value, all you need to do is add the object parameter inside curly braces as shown below. It is called a template directive and the syntax is specific to the Go templating

{{ .Object.Parameter }}
First Let’s understand what is an Object. Following are the three Objects we are going to use in this example.

Release: Every helm chart will be deployed with a release name. If you want to use the release name or access release-related dynamic values inside the template, you can use the release object.
Chart (chart.yaml): If you want to use any values you mentioned in the values.yaml, you can use the chart object.
Values (values.yaml): All parameters inside values.yaml file can be accessed using the Values object.

![image](https://user-images.githubusercontent.com/12657295/251477565-9c947e9e-45a2-440d-9916-e715912f03bc.png)


container name: {{ .Chart.Name }}: For the container name, Chart object is used  and use the chart name from the chart.yaml as the container name.
Replicas: {{ .Values.replicaCount }}  access the replica value from the values.yaml file.
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}" Multiple template directives are used in a single line and accessing the repository and tag information under the image key from the Values file.


Templated  
deployment.yaml file after applying the templates. The templated part is highlighted in bold. Replace the deployment file contents with the following.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
  labels:
    app: nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP


Service.yaml file 

apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  selector:
    app.kubernetes.io/instance: {{ .Release.Name }}
  type: {{ .Values.service.type }}
  ports:
    - protocol: {{ .Values.service.protocol | default "TCP" }}
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}

pipe ( | )  is used to define the default value of the protocol as TCP.


configmap.yaml and add the following contents to it.  the default Nginx index.html page  is replaced with a custom HTML page. Also,  a template directive  is added to replace the environment name in HTML.

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-index-html-configmap
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi! deployement in {{ .Values.env.name }} Environment using Helm Chart </h1>
    </html>

values.yaml
The values.yaml file contains all the values that need to be substituted in the template directives we used in the template

place the default values.yaml content with the following.

replicaCount: 2
image:
  repository: nginx
  tag: "1.16.0"
  pullPolicy: IfNotPresent
service:
  name: nginx-service
  type: ClusterIP
  port: 80
  targetPort: 9000
env:
  name: dev




helm install frontend nginx-chart

w a single helm chart can be used for multiple environments using different values.yaml files. To install a helm chart with external values.yaml file, you can use the following command with the --values flag and path of the values file.

$helm install frontend nginx-chart --values env/prod-values.yaml
When you have Helm as part of your CI/CD pipeline, you can write custom logic to pass the required values file depending on the environment.

if  the version is needed to roll back to the specific version, you can put the revision number like this.

helm rollback <release-name> <revision-number>
For example,

$helm rollback frontend 2

Uninstall The Helm Release
To uninstall the helm release use uninstall command. It will remove all of the resources associated with the last release of the chart.

$helm uninstall frontend
We can package the chart and deploy it to Github, S3, or any other platform.

$helm package frontend



