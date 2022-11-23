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
  
#### Step 7: Once docker is installed, we have to clone the repository from github and create Docker file for build the image and push to dockerhub:

- Once you have clone the repository from github, Create the dockerfile as like below:
  
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

- In this first we have to add resource limit to application pod enable metrics. So, we need to add below lines in nodeapp.yaml file.

  ```
  resources:
    requests:
      memory: "64Mi"
      cpu: "250m"
    limits:
      memory: "128Mi"
      cpu: "500m"
  ```
 
- For enable metrics run below command 
  ```minikube addons enable metrics```
  
  
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
  
- Now run below command and check if target is greater than utilization then it will create more replicas or not 
  
  ```kubectl get hpa```
  
#### Step 12: Add Health probes

- For this we need to add below lines in nodeapp.yaml file.

  ```
   readinessProbe:
     tcpSocket:
       port: 3000
     initialDelaySeconds: 5
     periodSeconds: 10
   livenessProbe:
     tcpSocket:
       port: 3000
     initialDelaySeconds: 15
     periodSeconds: 20
  ```
  
- In this we are using TCP liveness probe,this liveness probe uses a TCP socket. With this configuration, the kubelet will attempt to open a socket to     your container on the specified port. If it can establish a connection, the container is considered healthy, if it can't it is considered a failure.

- We are using both readiness and liveness probes in this liveness probe.

- In our scenario, The kubelet will send the first readiness probe 5 seconds after the container starts. This will attempt to connect to the application   container on port 3000. If the probe succeeds, the Pod will be marked as ready. The kubelet will continue to run this check every 10 seconds.

- In addition to the readiness probe, this configuration includes a liveness probe. The kubelet will run the first liveness probe 15 seconds after the     container starts. Similar to the readiness probe, this will attempt to connect to the application container on port 3000. If the liveness probe fails,   the container will be restarted.

- For the SSL minikube already install self-signed certificate so not required to install again.

#### Step 13:  kustomization

- For this we have to build a file in kube directories as like below with the name kustomization.yaml for including the yaml files in kustomization. 
  
  ```
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization
  resources:
  - nodeapp.yaml
  - nodeappingress.yaml
  - mongo.yaml
  - appservice.yaml
  - hpapod.yaml
  - mongopvc.yaml
  - mongosvc.yaml

  ```
  
- Kustomize is a configuration management solution that leverages layering to preserve the base settings of your applications and components by             overlaying declarative yaml artifacts (called patches) that selectively override default settings without actually changing the original files.
  
- Now we have to work on outside the kube directory for override the existing application files. It's not changed the original files.

- For this we have to create the the kustomization file firt as like below with the name of kustomization.yaml
 
  ```
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  bases:
  - ./kube/

  patchesStrategicMerge:
    - replicas-patch.yml
  
  ```
- After that we can create replicas-patch.yml for override the existing content.
 
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nodeapp
  spec:
    replicas: 2
    
  Now it will create the 2 application pod previously it creates 1.
  
- For run the kustomization we have to run the below command outside the kube directory as like below:

  ```kubectl apply -k . ```
  
- Once command successfully run you got the 2 application pod in running state rather than one due to kustomization.

#### Step 14: Install Jenkins and configure pipeline:

- In this we need to create the deployment pod for jenkins and create PV for storing the data as like below file with the name jenkins.yaml

  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata: 
    name: jenkin-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 260Mi
  ---

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: jenkins
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: jenkins
    template:
      metadata:
        labels:
          app: jenkins
      spec:
        containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          ports:
            - name: http-port
              containerPort: 8080
            - name: jnlp-port
              containerPort: 50000
          volumeMounts:
            - name: jenkins-vol
              mountPath: /var/jenkins_home
        volumes:
          - name: jenkins-vol
            persistentVolumeClaim:
              claimName: jenkin-pvc
  ```            

- Now we need to create two services for jenkins as like below in single file with the name jenkins-service.yaml:
  
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: jenkins
  spec:
    type: NodePort
    ports:
      - port: 8080
        targetPort: 8080
        nodePort: 30000
    selector:
      app: jenkins

  ---

  apiVersion: v1
  kind: Service
  metadata:
    name: jenkins-jnlp
  spec:
    type: ClusterIP
    ports:
      - port: 50000
        targetPort: 50000
    selector:
      app: jenkins
   ```
- Jenkins runs on Tomcat, which uses port 8080 as the default. -p 5000:5000 required to attach slave servers; port 50000 is used to communicate between     master and slaves that's why we have created two services.

- Now we have to run both the file with apply commands as like below:

  ```kubectl apply -f jenkins.yaml```
  ```kubectl apply -f jenkins-service.yaml```

- After that we need to check pods and services are running fine related to jenkins from below command:
  
  For pods
  
  ```kubectl get pods```
  
  For services
  
  ```kubectl get svc```
  
- Now we have to setup the jenkins 

- To set up your installation, visit Jenkins on 30000 port, using your server domain name or IP address: http://your_server_ip_or_domain:30000.
  
  You should receive the Unlock Jenkins screen, which displays the location of the initial password:
  
  ![image](https://user-images.githubusercontent.com/42695637/190149938-d9a9b447-312b-40a3-bc1a-efdbdf0534c0.png)
  
- For this you have to enter inside the minikube machine with ssh, command is ```minikube ssh``` and after that with docker command, we need to enter       inside the jenkin container, After that use cat command to display the password in jenkin container:
  
  ```sudo cat /var/lib/jenkins/secrets/initialAdminPassword```
  
- Copy the 32-character alphanumeric password from the jenkin container and paste it into the Administrator password field in jenkin setup, then click     Continue.

- The next screen presents the option of installing suggested plugins or selecting specific plugins:
  
  ![image](https://user-images.githubusercontent.com/42695637/190150451-8d2339d0-be7b-43af-b367-b575ff553058.png)
  
  We’ll click the Install suggested plugins option if we want otherwise cancel the same and add plugins according to requirement like ssh for               authentication of manage nodes, which will immediately begin the installation process.  

- When the installation is complete, you’ll be prompted to set up the first administrative user. It’s possible to skip this step and continue as admin     using the initial password, So I have skip this and continue as admin, Now we got jenkin installtion is successfully completed.

- Once you got prompt of dashboard in jenkins, first change the password of admin through manage credentials because don't get password in setup.

- Now, we have to install SSH plugin for connectivity between master node and slave node with SSH.

- After that we have to add local system ssh credentials, so that it will become a slave and run the kubectl command through jenkins. 

- Now, we have to click on manage node in jenkin and click on new node
  
  ![image](https://user-images.githubusercontent.com/42695637/190152554-c2312226-aeda-4bdf-93c4-1898ec3354c7.png)
  
- Give the name and choose permanent agent. After that click on create
   
  ![image](https://user-images.githubusercontent.com/42695637/203483714-c497ff90-64c6-4944-942e-8949d6c72146.png)

- Now fill the details like below:
  
  ![image](https://user-images.githubusercontent.com/42695637/203483972-fbd48990-3e31-43d7-b77c-f4d3d2a929ba.png)
  
  ![image](https://user-images.githubusercontent.com/42695637/203484063-37c1450d-d47c-4b2c-ba9a-b5568b5c3073.png)

  ![image](https://user-images.githubusercontent.com/42695637/203484142-44f7e999-56b5-4876-ac35-3045f67b7bb3.png)

- Now click on New Item for creating the Job in dashboard of Jenkins and choose Pipeline and click on ok and choose GitHub hook trigger for GITScm         polling for GitHub Plugin triggers a one-time polling on GITScm. When GITScm polls GitHub, it finds that there is a change and initiates a build 
  
  ![image](https://user-images.githubusercontent.com/42695637/190156166-9508073a-9273-4042-a783-736409337957.png)
  
- We have to give below option, Pipeline script from SCM and choose the file from github with the name Jenkinsfile  

  ![image](https://user-images.githubusercontent.com/42695637/203485253-152a4647-fbcc-4b62-8fa1-a94ecad564ca.png)

  ![image](https://user-images.githubusercontent.com/42695637/203485306-fe49675f-4cc6-4c26-a345-317e2ceb3ab0.png)

  ![image](https://user-images.githubusercontent.com/42695637/203485359-460f4da5-4906-447a-a49a-62ae34ff7112.png)

- Now on the local system, we have to install jdk for connectivity with jenkins

  Install the java on server with below command:
  
  ```apt-get install openjdk-11-jdk```
  
- Once all done you got slave node in sync as like below:
 
  ![image](https://user-images.githubusercontent.com/42695637/203486152-c8b0e9be-8993-4226-8ae7-842a6b34efdc.png)

- Now we have to add a Jenkin file in github from where we fetch the code. After that add the code as like below in Jenkin file to run the pipeline.
  
  ``` 
  pipeline {


    environment {
 
      imagename = "mak1993/nodeapp01"
      registryCredential = 'mak1993'
      dockerImage = ''
    }

  agent any

  stages {
    stage('Cloning Git') {
      steps {
        git([url: 'https://github.com/mayankkh1/minikube01.git', branch: 'main'])
 
        }
      }

   stage('Building image') {
        steps{
          script {
            dockerImage = docker.build imagename
          }
        }
      }
  stage('Deploy Image') {
        steps{
          script {
            docker.withRegistry( '', registryCredential ) {
              dockerImage.push("$BUILD_NUMBER")
            
            }
          }
        }
      }

  stage('Deploy to Server') {
      steps{
        
        sh "sed -i 's/nodeapp01:latest/nodeapp01:$BUILD_NUMBER/g' kube/nodeapp.yaml" 
        sh "kubectl apply -k ."
   
           
          }  
       }

    }

  }
  
  ```

- For pushing the image to dockerhub, We have to add the credentials related to dockerhub in manage credentials in Jenkin dashboard.
  
- Once the credentials added, Now we are able to push the docker image to dockerhub.

- In this pipeline, we are trying to run the kustomization through kubectl command, As we have a application deployment file, In this file we have         mentioned specific image which is present in dockerhub. For example: mak1993/nodeapp01:latest

- Once the pipeline will run, it will start the pods and services but if any changes occur in image it will not reflect on website because it will not 
  consider a new change as name is same.
  
- So we are using the sed to change the specific image name with the buildnumber and also we are push the image with build number only.

- We can use below steps also rather than sed in Jenkin file but for this we need kubernetes deployment plugin, right now it's vulnerable so we are using   sed.

  ```
  kubernetesDeploy (configs: "kube/nodeapp.yaml", kubeconfigId: "Kubernetes", enableConfigSubstitution: "true")
  ```
  
  Also, for this we need to add Build_Number as a Variable rather than latest in nodeapp file, So it will get the build number directly in nodeapp yaml     file. 
  
  ```
  mak1993/nodeapp01:${BUILD_NUMBER}
  ```

#### Step 15: Add Github webhook URL
  
   








          

  


