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

- First, download the latest Minikube binary using the wget command:

  ```wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64```

- Copy the downloaded file and store it into the /usr/local/bin/minikube directory with:

  ```sudo cp minikube-linux-amd64 /usr/local/bin/minikube```

- Next, give the file executive permission using the chmod command:

  ```sudo chmod 755 /usr/local/bin/minikube```

- Finally, verify you have successfully installed Minikube by checking the version of the software:

  ```minikube version```

  The output should display the version number of the software, as in the image below.
  
  ![image](https://user-images.githubusercontent.com/42695637/203239634-f459d23e-90ed-4f14-ba88-9b41e8adfdd1.png)

#### Step 4: Install Kubectl

To deploy and manage clusters, we need to install kubectl, the official command line tool for Kubernetes:

- Download kubectl with the following command:
  
  ```curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl```
  
- Make the binary executable by typing:
  
  ```chmod +x ./kubectl```
  
- Then, move the binary into your path with the command:
   
  ```sudo mv ./kubectl /usr/local/bin/kubectl```
  
- Verify the installation by checking the version of your kubectl instance with below command:
  
  ```kubectl version -o json```
  
####  Step 5: Start Minikube  

- Once we have set up all the required software, we are ready to start Minikube with the below command: 

  ```minikube start```
  
#### Step 6: Now we have to install docker for build the nodejs image

- Update the apt package index and install packages to allow apt to use a repository over HTTPS: 
 
  ```sudo apt update``` 
  
  ``` sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release```
  
- Add Docker’s official GPG key:    
  
  ```sudo mkdir -p /etc/apt/keyrings```
  ```curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg```
  
- Use the following command to set up the repository:

  ```echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null```
  
- For install the Docker Engine update the apt package index again:
 
  ```sudo apt-get update```
  
- Now install docker with below command:
  
  ```sudo apt-get install docker-ce docker-ce-cli containerd.io```
  
#### Step 7: Once docker is installed, we have to pull the code from github and create Docker file for build the image:

- Once you have pull the code from github, Create the dockerfile as like below 
  
  ```FROM node:12.0-slim
  COPY . .
  RUN npm install
  CMD [ "node", "index.js" ]```

  


