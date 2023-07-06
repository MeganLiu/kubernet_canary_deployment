Validate the Helm Chart
 to make sure that our chart is valid and, all the indentations are fine, we can run the below command.
 Ensure you are inside the chart directory.

helm lint .
If you are executing it from outside the nginx-chart directory, provide the full path of nginx-chart

helm lint /path/to/nginx-chart

To validate if the values are getting substituted in the templates, you can render the templated YAML files with the values using the following command. It will generate and display all the manifest files with the substituted values.

helm template .
We can also use --dry-run command to check. This will pretend to install the chart to the cluster and if there will be some issue it will show the error.

helm install --dry-run my-release nginx-chart
