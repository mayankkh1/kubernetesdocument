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
  
  ```
  sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
  ```   
  
- Add Docker’s official GPG key:    
  
  ```sudo mkdir -p /etc/apt/keyrings```
  ```curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg```
  
- Use the following command to set up the repository:

  ```
  echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
  ```
  
- For install the Docker Engine update the apt package index again:
 
  ```sudo apt-get update```
  
- Now install docker with below command:
  
  ```sudo apt-get install docker-ce docker-ce-cli containerd.io```
  
#### Step 7: Once docker is installed, we have to pull the code from github and create Docker file for build the image and push to dockerhub:

- Once you have pull the code from github, Create the dockerfile as like below:
  
  ```
  FROM node:12.0-slim
  COPY . .
  RUN npm install
  CMD [ "node", "index.js" ]
  
  ```

- Now we have to build the image with below command:

  ```docker build -t mak1993/nodeapp01:latest . ```
  
- Once image is build now we need to push to dockerhub.

  ```docker push mak1993/nodeapp01:latest```
 
  Notes: For push the docker image first we need to log in to Docker Hub from our command-line interface then run above command.
  
#### Step 8: Now we have to make deployment files for application and database:

- Create the application file for deployment as like below with the name nodeapp.yaml:
  
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nodeapp
  spec:
    selector:
      matchLabels:
        app: nodemyapp01
    template:
      metadata:
        labels:
          app: nodemyapp01
      spec:
        containers:
          - name:  nodeapp
            image: mak1993/nodeapp01:latest
            ports:
              - containerPort: 3000
            env:
              - name: MONGO_URL
                value: mongodb://mongo:27017/dev
            imagePullPolicy: Always
     ```     
     
- Now we need to create database file for deployment. In this we need to create PVC first then create the deployment file
  
  Create the PersistentVolumeClaim as like below with the name mongopvc.yaml:
  
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: mongo-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 256Mi
  ```
  After the PVC file, Now create the deployment file as like below with the name mongo.yaml:

  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mongo
  spec:
    selector:
      matchLabels:
        app: mongo
    template:
      metadata:
        labels:
          app: mongo
      spec:
        containers:
          - name: mongo
            image: mongo:3.6.17-xenial
            ports:
              - containerPort: 27017
            volumeMounts:
              - name: storage
                mountPath: /data/db
        volumes:
          - name: storage
            persistentVolumeClaim:
              claimName: mongo-pvc
  ```
  
- Once all files are created, Put the files in any folder(in my case I have uploaded in kube folder).

- Now run the below command kubectl apply for creating the deployment pods.

  ```kubectl apply -f kube```
  
- After that check the deployment pods are working fine or not with the below command.

  ```kubectl get pods```
              
- Once you get the pods in running state then your deployments files are working fine.

#### Step 9: Now we have to expose this nodejs app to outside this cluster.

- For this we need to create a service manifest file for both pod database and nodejs.

  Create the Database service file as like below with the name mongosvc.yaml:
  
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: mongo
  spec:
    selector:
      app: mongo
    ports:
      - port: 27017
        targetPort: 27017
  ```
  Create the App service file with type NodePort as like below with the name appservice.yaml:
  
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: nodeapp
  spec:
    selector:
      app: nodemyapp01
    ports:
      - port: 8080
        targetPort: 3000
    type: NodePort
  ```
  
- Put both the files in kube folder as like previous file and now we have to run again the below command for creating the service
  
  ```kubectl apply -f kube```
  
- After that check service is succesfully created or not using below command:

  ```kubectl get svc```   
  
- Now we can check the nodeport url is working fine or not. Once Url is working fine we can move forward.

#### Step 10: Create Ingress Controller and ingress file for pointing to domain

- To enable the NGINX Ingress controller, run the following command in minikube:
  
  ```minikube addons enable ingress```
  
- Verify that the NGINX Ingress controller is running with the below comman:

  ```kubectl get pods -n ingress-nginx```
  
- If you want to add controller manually as like in k3s, run below command:
  
  ```kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/baremetal/deploy.yaml```
  
- Now we have to create the ingress file for domain on 80 or 443 port points to our deployment application

  Create the ingress file as like below with the name nodeappingress.yaml and put the file into kube folder 
  
  ```
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: nodeappingress
  spec: 
    ingressClassName: nginx
    rules:
    - host: "mywebapp.org"
      http:
        paths:
        - pathType: Prefix
          path: /
          backend:
            service:
              name: nodeapp
              port: 
                number: 8080
 
  ```
  
- We have to run the below command for creating the ingress resource
  
  ```kubectl apply -f nodeappingress.yaml ```

- Now check ingress resource is created or not, run below command and check address is assigned or not.
 
  ```kubectl get ingress```

  ![image](https://user-images.githubusercontent.com/42695637/203298947-fceef6f9-b7de-4566-bcae-0f69902f3aa7.png)

- Now run this "mywebapp.org" URL on browser, You got your website content in browser after hit the URL.

#### Step 11: Create manifest file for Horizontal Pod Autoscaling.

- In this first we have to add resource limit to application pod. So, we need to add below lines in nodeapp.yaml file.

  ```
  resources:
    requests:
      memory: "64Mi"
      cpu: "250m"
    limits:
      memory: "128Mi"
      cpu: "500m"
  ```
  
- Now for HorizontalPodAutoscaling we have to create file with the name hpapod.yaml and put the file into kube folder 

  ```
  apiVersion: autoscaling/v2
  kind: HorizontalPodAutoscaler
  metadata:
    name: nodeapphorizontal
  spec:
    maxReplicas: 4
    metrics:
    - resource:
        name: cpu
        target:
          averageUtilization: 1
          type: Utilization
      type: Resource
    minReplicas: 1
    scaleTargetRef:
      apiVersion: apps/v1
      kind: Deployment
      name: nodeapp 
     
  ```
  
- We have to run the below command for creating the HorizontalPodAutoscaler
 
  ```kubectl apply -f hpapod.yaml```
  
- 
  
  
  
   








          

  


