# Deploying a Simple Flask App on Minikube (Step‑by‑Step Guide)

### Pre-requisite

- Docker Desktop to run your minikube as a container or a virtual machine environment to run it as a Virtual machine.
- Minikube cluster
- Web application Docker image to be pulled in the kubernetes cluster
- VSCode
- Make sure you have enough resources on your local machine and refer to the minikube official documentation [Here](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download) to install minikube depending on your operating system. 


### Start your Minikube Cluster

Once minikube is installed, open your terminal with administrator access and run the following command to start minikube as a Docker container:

`minukube start --driver docker`

The output should be as seen in the image below 

![minikube cluster start](image.png)

### Check the status of the cluster

To verify that all the components are running run the command:

`minikube status`

![minikube cluster status](image-1.png)

### To display the nodes in the cluster  

To interact with the cluster we will need Kubectl which is already installed as a dependency when  minikube is installed.

Run the command to display the nodes in the cluster. 

`kubectl get node`

### Create a project in VSCode and create basic deployment and service configuration files in yaml

Run the follwoing command to check the pods in the default namespace. None have been created yet.

`kubectl get pod` 


1. A Deployment to manage the pods
   
Refer to the Kubernetes Documentation [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) to create a deployment configuration file in yaml using the following example and then save the file in your project folder.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  labels:
    app: flask-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: your_docker_image:tag
        ports:
        - containerPort: 5000
```

Run the command:

`kubectl apply -f <deployment-name.yaml>` 

Verify the deployment by running:

`kubectl get deployments` or `kubectl describe deployment <deployment-name>` for more details

1. A Service to expose the pods to the Internet

Refer to the Kubernetes Documentation [here](https://kubernetes.io/docs/concepts/services-networking/service/) to create a service configuration file in yaml using the following example and then save the file in your project folder.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  type: NodePort
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30000
```

Run the command:

`kubectl apply -f <service-name.yaml>`

### Interact with the cluster

Run the following command to check all the components in the cluster:

`kubectl get all` 

![all components](image-3.png)

Run the following command to get the node IP:

`kubectl get nodes -o wide`

Then copy and paste the INTERNAL-IP address followed by the node port to access the app on your browser.

`<node-id>:<nodePort>`

### Troubleshooting

On Windows + WSL2, NodePort cannot be accessed vis minikube IP. Run the following command to use minikube service instead, this will open a TCP tunnel to expose the port to Windows:

`minikube service <service-name>`

The output will be as seen below

![service url](image-4.png)

Press Ctrl + click on the URL to access the app on your browser.

![app on browser](image-5.png) 

Press Ctrl + C to stop the tunnel

### Cleanup the cluster

`kubectl delete -f <deployment-name.yaml>`

`kubectl delete -f <service-name.yaml>`

Open your terminal in administrator mode and run `minikube delete` to delete minikube container
