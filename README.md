Kubernetes Fundamentals: Deploying a Stateless NGINX Web Server

![alt text](https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png)

This project is a hands-on guide to mastering the fundamental Kubernetes workflow. We will take a standard NGINX container image, deploy it with redundancy on a local Kubernetes cluster, and expose it to be accessible from a web browser.

This project focuses on the core, stateless building blocks of Kubernetes, providing a solid foundation for more advanced concepts.
🎯 Objective

To deploy a highly available, stateless web server on Kubernetes and make it accessible. By the end of this project, you will have a practical understanding of how to run and manage applications the "Kubernetes way."
✨ Core Concepts Covered

    kubectl: The command-line interface for Kubernetes.

    YAML Manifests: The declarative way to define Kubernetes objects.

    Pods: The smallest deployable units in Kubernetes.

    Deployments: Managing application lifecycle, replicas, and updates.

    ReplicaSets: Ensuring a specified number of Pod replicas are running.

    Labels & Selectors: The glue that connects different Kubernetes resources.

    Services: Providing a stable network endpoint for your application.

    Service Types: Specifically NodePort for external access in a local environment.

🛠️ Prerequisites

Before you begin, ensure you have the following installed and configured:

    A Local Kubernetes Cluster:

        minikube (Recommended for this guide)

        kind

        Docker Desktop with Kubernetes enabled

    kubectl: The Kubernetes command-line tool. Installation Guide.

    A Text Editor: Such as VS Code, Sublime Text, or vim.

    A Web Browser.

🚀 Project Steps
Step 1: Start Your Cluster

First, make sure your local Kubernetes cluster is running. For minikube, use the following command:
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

minikube start

Verify that your cluster is ready by listing the nodes:
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

kubectl get nodes

# EXPECTED OUTPUT:

# NAME STATUS ROLES AGE VERSION

# minikube Ready control-plane ... ...

Step 2: Create the Kubernetes Manifest

We will define all our Kubernetes resources in a single YAML file. Create a file named my-webapp.yaml.

This file will contain two objects separated by ---: a Deployment to manage our application Pods, and a Service to expose them.
code Yaml
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

# my-webapp.yaml

# ===================================================================

# Deployment: The blueprint for our application instances (Pods)

# ===================================================================

apiVersion: apps/v1
kind: Deployment
metadata:
name: my-webapp
spec:

# We want 3 identical copies of our application running for high availability

replicas: 3
selector:
matchLabels: # This selector tells the Deployment which Pods to manage. # It must match the labels in the Pod template below.
app: my-webapp
template:
metadata:
labels: # These labels are applied to each Pod created by this Deployment. # This is how the Service will find them.
app: my-webapp
spec:
containers: - name: nginx-container
image: nginx:1.21
ports: - containerPort: 80 # NGINX listens on port 80 inside the container

---

# ===================================================================

# Service: The stable network endpoint for our Pods

# ===================================================================

apiVersion: v1
kind: Service
metadata:
name: my-webapp-service
spec:

# 'NodePort' makes the service accessible from outside the cluster

# on a static port on the Node's IP address.

type: NodePort
selector: # This selector is the crucial link. It directs traffic to any Pod # that has the 'app: my-webapp' label.
app: my-webapp
ports: - protocol: TCP # Port 80 is the port on the Service itself.
port: 80 # Port 80 is the targetPort on the Pods.
targetPort: 80

Step 3: Deploy and Inspect the Application

Apply the manifest to your cluster. This tells Kubernetes to create the resources defined in the file.
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

kubectl apply -f my-webapp.yaml

Now, let's verify that everything was created successfully.
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

# Check that the Deployment is created and all replicas are ready

kubectl get deployments

# EXPECTED OUTPUT:

# NAME READY UP-TO-DATE AVAILABLE AGE

# my-webapp 3/3 3 3 ...

# Check that 3 Pods are running

kubectl get pods

# EXPECTED OUTPUT:

# NAME READY STATUS RESTARTS AGE

# my-webapp-7c64c7485f-abcde 1/1 Running 0 ...

# my-webapp-7c64c7485f-fghij 1/1 Running 0 ...

# my-webapp-7c64c7485f-klmno 1/1 Running 0 ...

# Get more details about the Deployment's status and events

kubectl describe deployment my-webapp

Step 4: Expose and Access the Application

The Service we created exposed our application. Let's find out where we can access it.
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

# Get information about our service, including the NodePort

kubectl get services

# EXPECTED OUTPUT:

# NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE

# my-webapp-service NodePort 10.108.144.20 <none> 80:3XXXX/TCP ...

# kubernetes ClusterIP 10.96.0.1 <none> 443/TCP ...

Note the port mapping under PORT(S). The 3XXXX number is the NodePort assigned by Kubernetes.

The easiest way to access the service via minikube is:
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

minikube service my-webapp-service

This command will automatically open the NGINX welcome page in your default browser. You have successfully deployed and exposed your application!
⚙️ Day-1 Operations: Managing Your App

A key benefit of Kubernetes is easy management. Here are some common operations.
Scaling the Application

Imagine traffic has increased and you need more capacity. You can scale from 3 to 5 replicas with a single command:
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

kubectl scale deployment my-webapp --replicas=5

Verify the new Pods were created:
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

kubectl get pods # You should now see 5 pods running!

Viewing Logs

To troubleshoot or monitor an application, you need to see its logs.
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

# First, get the name of a pod

POD_NAME=$(kubectl get pods -l app=my-webapp -o jsonpath='{.items[0].metadata.name}')

# Then, view its logs

kubectl logs $POD_NAME

🧹 Cleanup

To remove all the resources we created, simply delete them using the same manifest file.
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

kubectl delete -f my-webapp.yaml

If you are done with the cluster, you can stop it to free up resources on your machine.
code Bash
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

minikube stop

🗺️ Architecture Overview

Here is a simple diagram of what we built:
code Code
IGNORE_WHEN_COPYING_START
IGNORE_WHEN_COPYING_END

INTERNET
│
▼
[ Minikube Node IP : NodePort ] (e.g., 192.168.49.2:31234)
│
▼
[ my-webapp-service ] (Acts as an internal load balancer)
│
├─► [ Pod 1 (NGINX) ]
│
├─► [ Pod 2 (NGINX) ]
│
└─► [ Pod 3 (NGINX) ]
