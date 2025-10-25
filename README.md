# Customize Your Helm Chart

## Understanding Helm Charts, Values, and Templates

### Why Helm Charts Are Needed
Helm charts are **packages** or collections of pre-configured Kubernetes resources.  
They simplify the deployment and management of applications in Kubernetes by bundling all necessary resources into a single, manageable unit.

Helm charts promote **reusability** and **consistency**, allowing you to define, install, and upgrade even the most complex Kubernetes applications easily.

---

### What Are Charts, Values, and Templates

**Charts:**  
A Helm chart is a directory with a predefined structure. It contains all the resource definitions needed to run an application, tool, or service inside a Kubernetes cluster. Think of it as a recipe with instructions on how to create and run a Kubernetes application.

**Values:**  
The `values.yaml` file inside a chart provides configuration values for a chart's templates. These values can be overridden during chart installation or upgrades, allowing for flexibility and customization without altering the chart's core logic.

**Templates:**  
The `templates/` directory contains the template files. These are standard Kubernetes YAML files with placeholders (`{{ .Values.someParameter }}`). Helm dynamically fills these placeholders with the values from the `values.yaml` file or override values provided during runtime. This allows you to customize the deployment without changing the actual YAML files.

---

## Explore the `webapp` Directory

Navigate to the `webapp` directory created by Helm. Inside, you'll find:

- **Chart.yaml:** Contains metadata about the chart such as name, version, and description.  
- **values.yaml:** Provides configurable values that Helm will inject into the templates. Here you set default configuration values.  
- **templates/:** Contains the template files that will generate Kubernetes manifest files. These templates reference the values defined in `values.yaml`.

---

## Modify `values.yaml`

Open `values.yaml` in a text editor and set the image to use the Nginx stable version:

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent
```

This configuration will deploy **two replicas** (`replicaCount: 2`) of the Nginx server.  
Save your changes.

---

## Customize `templates/deployment.yaml`

Open the `deployment.yaml` file in the `templates/` directory.

Remove the line below from under `spec.template.spec.containers.resources`:

```yaml
{{- toYaml .Values.resources | nindent 12 }}
```

Then add a simple resource request and limit under `spec.template.spec.containers.resources`. This helps Kubernetes manage resources efficiently:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

These settings specify that your deployment should request **128Mi of memory** and **100m of CPU**, but it won't use more than **256Mi of memory** and **200m of CPU**.  
Save the file after making your changes.

---

## Commit and Push Changes

```bash
git add .
git commit -m "Customized Helm chart"
git push
```

---

## Step 2: Deploying Your Application

### Deploy with Helm

Navigate to the root of your project directory `helm-web-app` and deploy the application on Kubernetes using the below command:

```bash
helm install my-webapp ./webapp
```
![helm install](./images/helm%20install%20my-webapp.png)

### Check Deployment

```bash
kubectl get deployments
```
![kubectl get deployments](./images/kubectl%20get%20deployments.png)
### Visit Application URL

Get the application URL by running these commands:

```bash
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=webapp,app.kubernetes.io/instance=my-webapp" -o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
kubectl --namespace default port-forward $POD_NAME 8081:$CONTAINER_PORT
```

Then visit **[http://127.0.0.1:8081](http://127.0.0.1:8081)** to use your application.

![Nginx application](./images/nginx%20application.png)

### Errors and Resolutions

1. K8s cluster unreachable when `helm install my-webapp ./webapp` was ran

![k8s cluster unreachable error](./images/k8s%20cluster%20unreachable%20error.png)

**Fix:** Launch Docker Destop and run the below from your CLI  
```
minikube start
kind create cluster
```
Then re-run
```
helm install my-webapp ./webapp
```
![Helm install](./images/helm%20install%20my-webapp.png)

2. Error while trying to get the application URL after running:
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=webapp,app.kubernetes.io/instance=my-webapp" -o jsonpath="{".items[0].metadata.name"}")

export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{".spec.containers[0].ports[0].containerPort"}")

kubectl --namespace default port-forward $POD_NAME 8081:$CONTAINER_PORT
```
`export` is a Linux/macOS shell command (used in Bash to set environment variables),
but PowerShell or Command Prompt doesn’t recognize it.

**Fix:** If you prefer using the Linux-style commands like export, install and use Git Bash or Windows Subsystem for Linux (WSL).  

3. Application not reachable on localhost:8081
![Application unreachable](./images/application%20not%20reachable.png)

 Run `kubectl get deployments` and `kubectl get pods` to know the status of the deployment and the pods.
![Deployment error](./images/deployment%20error.png)
![Pods error](./images/ImagePullBackOff%20error.png)

Run `kubectl describe pod "$POD_NAME"` and scroll to the bottom of the output and look at the Events section to see the actual reason for the error.
![Fialed to pull image](./images/failed%20to%20pull%20image.png)

**Fix:** Ensure you have good internet connection. Check deployments and pods status again.
![Deployment and pods status](./images/pods%20and%20deployment%20running.png) 

Re-run the command to get the application URL.
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=webapp,app.kubernetes.io/instance=my-webapp" -o jsonpath="{".items[0].metadata.name"}")

export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{".spec.containers[0].ports[0].containerPort"}")

kubectl --namespace default port-forward $POD_NAME 8081:$CONTAINER_PORT
```
![Application URL](./images/port%20forwarding.png)
![](./images/nginx%20application.png)

