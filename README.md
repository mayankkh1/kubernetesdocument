## Description:

- Setup Minikube Kubernetes Server
- Create Node JS Deployment and Service
- PV & Storage class
- Create Ingress Controller and Access Above created deployment using Ingress and point it to the domain (application should be accessible through a browser) 
- HPA & Probs
- SSL certificate
- kustomization
- Jenkins declarative pipeline to deploy on K8s cluster

#### Step-1:  Install Minikube on server:

- Before installing any software, we need to update the system. To do so, run the below command:

```sudo apt-get update -y```

- Also, make sure to install (or check whether you already have) the following required packages:
 
```sudo apt-get install curl```

```sudo apt-get install apt-transport-https```

#### Step 2: As minikube is installing on a virtual machine in which we can set up our single node cluster with Minikube. So we need to install VirtualBox Hypervisor
  
- To install VirtualBox on Ubuntu, run the command:

```sudo apt install virtualbox virtualbox-ext-pack```

#### Step 3: Install Minikube 

-  First, download the latest Minikube binary using the wget command:

```wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64```

- Copy the downloaded file and store it into the /usr/local/bin/minikube directory with:

```sudo cp minikube-linux-amd64 /usr/local/bin/minikube```

-  Next, give the file executive permission using the chmod command:

```sudo chmod 755 /usr/local/bin/minikube```

- Finally, verify you have successfully installed Minikube by checking the version of the software:

```minikube version```

  The output should display the version number of the software, as in the image below.
  
  
