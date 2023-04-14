# Spring Boot based Java web application using Maven, SonarQube, Argo CD and Kubernetes

![Screenshot 2023-03-28 at 9 38 09 PM](https://user-images.githubusercontent.com/43399466/228301952-abc02ca2-9942-4a67-8293-f76647b6f9d8.png)

This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml 
at the root directory of the repository.

This is a MVC architecture based application where controller returns a page with title and message attributes to the view.


Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD and Kubernetes:

Prerequisites:

   -  Java application code hosted on a Git repository
   -  Jenkins server, SonarQube server and Minikube server for ArgoCD
   -  Argo CD

Tools Required:

   -  Java
   -  Git
   -  Maven
   -  Jenkins
   -  Docker
   -  SonarQube
   -  Minikube
   -  ArgoCD

For this project am launching two Amazon Linux 2 EC2 instances (Jenkins[t2.medium], SonarQube[t2.small]) and one ubuntu (K8s) using Virtual Box 

### Configuring Jenkins server

Pre-Requisites:

   -  Java
   -  Git
   -  Maven
   -  Docker
   -  Jenkins

Install Java

```shell
sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
java -version
```

Install Git

```shell
sudo yum install git -y
git --version
```

Install Maven

```shell
sudo wget https://dlcdn.apache.org/maven/maven-3/3.8.8/binaries/apache-maven-3.8.8-bin.tar.gz
sudo tar zxvf apache-maven-3.8.8-bin.tar.gz
cd apache-maven-3.8.8-bin
sudo yum install maven -y
mvn --version
```

Install Docker

```
sudo yum update -y
sudo amazon-linux-extras install docker -y

sudo sysytemctl enable docker
sudo sysytemctl start docker
sudo sysytemctl status docker
```

Install Jenkins

```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade -y
# Add required dependencies for the jenkins package
sudo yum install jenkins -y
jenkins --version

sudo systemctl enable jenkins.service
sudo systemctl start jenkins.service
```

Grant Jenkins user and ec2-user user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ec2-user
systemctl restart docker
```

As we installed many tools, its better to restart the server

```
sudo reboot now
```

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">


### Login to Jenkins using the below URL:

`http://<ec2-instance-public-ip-address>:8080`  [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
   -  1. Delete the inbound traffic rule for your instance
   -  2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      
<img width="1291" alt="Screenshot 2023-02-01 at 10 56 25 AM" src="https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png">

Click on Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 58 40 AM" src="https://user-images.githubusercontent.com/43399466/215959294-047eadef-7e64-4795-bd3b-b1efb0375988.png">

Wait for the Jenkins to Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 59 31 AM" src="https://user-images.githubusercontent.com/43399466/215959398-344b5721-28ec-47a5-8908-b698e435608d.png">

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]

<img width="990" alt="Screenshot 2023-02-01 at 11 02 09 AM" src="https://user-images.githubusercontent.com/43399466/215959757-403246c8-e739-4103-9265-6bdab418013e.png">

Jenkins Installation is Successful. You can now starting using the Jenkins 

<img width="990" alt="Screenshot 2023-02-01 at 11 14 13 AM" src="https://user-images.githubusercontent.com/43399466/215961440-3f13f82b-61a2-4117-88bc-0da265a67fa7.png">

Install the Required plugins in Jenkins

   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for `Docker Pipeline`, `SonarQube Scanner`.
   - Select the plugins and click the Install button.
   - Restart Jenkins after the plugin is installed. `http://<ec2-instance-public-ip-address>:8080/restart`
   
<img width="1392" alt="Screenshot 2023-02-01 at 12 17 02 PM" src="https://user-images.githubusercontent.com/43399466/215973898-7c366525-15db-4876-bd71-49522ecb267d.png">

Wait for the Jenkins to be restarted.

## Execute the application locally and access it using your browser

Checkout the repo and move to the directory

```
git clone https://github.com/iamsaikishore/Project3---java-maven-sonar-argocd-k8s.git
cd Project3---java-maven-sonar-argocd-k8s/sprint-boot-app
```

Execute the Maven targets to generate the artifacts

```
mvn clean package
```

The above maven target stroes the artifacts to the `target` directory. You can either execute the artifact on your local machine
(or) run it as a Docker container.

** Note: To avoid issues with local setup, Java versions and other dependencies, I would recommend the docker way. **


### Execute locally (Java 11 needed) and access the application on http://localhost:8080

```
java -jar target/spring-boot-web.jar
```

### The Docker way

Build the Docker Image

```
docker build -t spring-boot-app:v1 .
```

```
docker run -d -p 8010:8080 -t spring-boot-app:v1
```

Hurray !! Access the application on `http://<ec2-instance-public-ip-address>:8010`
   
![Screenshot (186)](https://user-images.githubusercontent.com/129657174/230657868-0cb2dff3-8038-4560-8475-ec855bcbbe23.png)

### Configuring SonarQube server

```
sudo -i
amazon-linux-extras install java-openjdk11 -y
adduser sonarqube
su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```
   
**Note: ** By default, SonarQube will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 9000 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 9000 as well, in my case, I allowed `All traffic`).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">


### Login to SonarQube using the below URL:

`http://<ec2-instance-public-ip-address>:9000`  [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
   -  1. Delete the inbound traffic rule for your instance
   -  2. Edit the inbound traffic rule to only allow custom TCP port `9000`

Hurray !! Now you can access the `SonarQube Server` on `http://<ec2-instance-public-ip-address>:9000` 
   
![Screenshot (199)](https://user-images.githubusercontent.com/129657174/230658262-0a0c8d3a-312d-4423-84e5-a7a1efa9fc68.png)

Login using username: admin, Passsword: admin and Change the password
   
![Screenshot (200)](https://user-images.githubusercontent.com/129657174/230658269-22ebd5d8-3f97-4a47-8155-57236a95b055.png)

Now at the right top corner click profile icon ->  My Account -> Security, Under Generate Token give a name and click Generate and copy the Token.
   
![Screenshot (201)](https://user-images.githubusercontent.com/129657174/230658498-75e5a06e-512e-4665-bd81-68d7a87d3469.png)
![Screenshot (202)](https://user-images.githubusercontent.com/129657174/230658495-a4ee14e9-df19-4bfa-8cec-0b9ccc3abb76.png)
   
### Configuring Credentials on Jenkins

For SonarQube

   -  Go to Manage Jenkins > Manage Credentials > System > global > Add Credentials
   -  Select Kind as Secret text
   -  Copy the Sonarqube Token in Secret box and give name as sonarqube in ID
   -  Click Save

For Git

   -  Now go to your GitHub Account > Settings > Developer Settings > Personal access tokens > Tokens(classic) > Generate new token (classic)
   -  Give a name, Select all check boxes, Click Generate token and Copy the token for future use
   -  Go to Manage Jenkins > Manage Credentials > System > global > Add Credentials
   -  Select Kind as Secret text
   -  Copy the Git Secret Token in Secret box and give name as github in ID
   -  Click Save

For DockerHub

   -  Now go to [DockerHub](https://hub.docker.com/) create a user and create a new repository with name 'spring-boot-app'
   -  Go to Manage Jenkins > Manage Credentials > System > global > Add Credentials
   -  Select Kind as Username and Password
   -  Give the username and password and give name as docker-cred in ID
   -  Click Save

### Configuring Minikube server

```
mkdir minikube && cd minikube
vim minikube.sh
```

Copy the following script

```shell
#!/bin/bash

sudo apt-get update -y
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io -y

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
####
echo the script is now ready
echo manually run minikube start --vm-driver=docker --cni=calico to start minikube

sudo usermod -aG docker $USER
newgrp docker

minikube start --vm-driver=docker --cni=calico
```

```
chmod u+x minikube.sh
./minikube.sh
```

```
minikube start
```

### Configure ArgoCD

Go to operatorhub.io, search for ArgoCD and click `Install` [Install Argo CD](https://operatorhub.io/operator/argocd-operator) 

   -  Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.

```
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.24.0/install.sh | bash -s v0.24.0
```

   -  Install the operator by running the following command:What happens when I execute this command?

```
$ kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
```

This Operator will be installed in the "operators" namespace and will be usable from all namespaces in the cluster.

   -  After install, watch your operator come up using next command.

```
$ kubectl get csv -n operators
```

The following example shows the most minimal valid manifest to create a new Argo CD cluster with the default configuration.

```
$ vim basic-argocd.yml

apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}
```

```
kubectl create -f basic-argocd.yml
kubectl get pods,svc
```
***Note***  It will take some time to bring up the pods

Once the pods are up change the ClusterIP service to Nodeport to access Argo CD UI

```
kubectl edit service example-argocd-service
```

Change `type: ClusterIP` to `type: NodePort`
   
```
   kubectl get po,svc
```
   
![Screenshot (188)](https://user-images.githubusercontent.com/129657174/230659731-b9fe3d55-ce9a-4925-be03-ace1c4a2efc0.png)

```
kubectl get svc
minikube service example-argocd-service
minikube service list
```
   
![Screenshot (187)](https://user-images.githubusercontent.com/129657174/230659894-4a517261-5f19-44bb-9989-1a411aaf3af6.png)
   
***Note***  As we are using Nodeport service we can access ArgoCD UI within the network

Now copy the ip-address of example-argocd-service, paste it in the browser, in my case am using virtual box
   
![Screenshot (189)](https://user-images.githubusercontent.com/129657174/230659974-7ff86b4a-76ee-4830-beb6-80917e43780b.png)

Username is 'admin' and for password

```
kubectl get secret
kubectl edit secret example-argocd-cluster
```

Copy the encoded secret

``
echo <encoded-secret> | base64 -d
``

![Screenshot (194)](https://user-images.githubusercontent.com/129657174/230660086-a6065949-6ba7-411d-b95a-5aa00b8d90d2.png)

![Screenshot (195)](https://user-images.githubusercontent.com/129657174/230665375-c04da1a1-8b1b-4947-85f7-2596a5465a49.png)
   
![Screenshot (196)](https://user-images.githubusercontent.com/129657174/230660105-0a2f81dc-396c-4544-9cb2-be6f8c49425d.png)

copy the decoded secret for password and login

![Screenshot (198)](https://user-images.githubusercontent.com/129657174/230660575-26bf787f-aff7-4aa4-8556-eee26ae6407d.png)
   
Click on ``CREATE APPLICATION``
   
![Screenshot (210)](https://user-images.githubusercontent.com/129657174/230662104-cf1864f3-c2db-4d8e-ad09-75a718feda99.png)

![Screenshot (207)](https://user-images.githubusercontent.com/129657174/230662167-b1ac35c0-5de5-4a40-8a76-575a99954f3d.png)
   
![Screenshot (208)](https://user-images.githubusercontent.com/129657174/230665046-983db740-2435-402f-8f5d-be4bad15c32a.png)
   
Now click ``CREATE``
   
![Screenshot (212)](https://user-images.githubusercontent.com/129657174/230662205-e2adaa8f-37a4-4d16-9a2f-cd9f3a5d2fd8.png)

## Jenkins Pipeline

```
git clone https://github.com/iamsaikishore/Project3---java-maven-sonar-argocd-k8s.git
cd Project3---java-maven-sonar-argocd-k8s/spring-boot-app
ls -l
```

Go through the JenkinsFile, Dockerfile, pom.xml

```
cat Dockerfile
# You can change this base image to anything else
# But make sure to use the correct version of Java
FROM adoptopenjdk/openjdk11:alpine-jre

# Simply the artifact path
ARG artifact=target/spring-boot-web.jar

WORKDIR /opt/app

COPY ${artifact} app.jar

# This should not be changed
ENTRYPOINT ["java","-jar","app.jar"]
```

This Dockerfile is used to build a Docker image for a Spring Boot web application using OpenJDK 11 on an Alpine Linux distribution.

The first line specifies the base image that will be used to build the Docker image. In this case, the image being used is "adoptopenjdk/openjdk11:alpine-jre", which is a pre-built image containing OpenJDK 11 on Alpine Linux.

The second line specifies an argument that will be used to pass the path of the application's artifact (JAR file) to the Docker build process.

The third line sets the working directory within the Docker container to "/opt/app".

The fourth line copies the application's artifact (specified by the "artifact" argument) to the Docker container and renames it to "app.jar".

The final line specifies the command that will be run when the Docker container is started. In this case, it runs the Java command to execute the "app.jar" file.

Overall, this Dockerfile sets up a simple and efficient environment for running a Spring Boot web application using Java 11 on Alpine Linux.

```
cat JenkinsFile
```

This is a Jenkins Pipeline script that automates the building, testing, analysis, and deployment of a Spring Boot web application using Maven, Docker, and Kubernetes.

The script defines a pipeline with multiple stages, each of which represents a step in the build process.

The agent section defines the Jenkins agent to be used in the pipeline. In this case, the agent is a Docker container based on the 'abhishekf5/maven-abhishek-docker-agent:v1' image. The 'args' option is used to mount the Docker socket on the host machine to the container so that it can access the Docker daemon on the host machine. The '--user root' option is used to run the container as the root user, which is required to perform certain privileged operations such as building Docker images.

The first stage, "Checkout", checks out the source code from a Git repository.

The second stage, "Build and Test", builds the project and creates a JAR file using Maven.

The third stage, "Static Code Analysis", performs static code analysis using SonarQube. The SonarQube server is specified using the "SONAR_URL" environment variable, and the authentication token is obtained using the "sonarqube" credential.

The fourth stage, "Build and Push Docker Image", builds a Docker image using the JAR file and the Dockerfile located in the "spring-boot-app" directory. The resulting Docker image is tagged with the build number and pushed to a Docker registry using the "docker-cred" credential.

The fifth stage, "Update Deployment File", updates the Kubernetes deployment file located in the "spring-boot-app-manifests" directory to use the newly built Docker image. The update is performed using a sed command, and the changes are committed to the Git repository using the "github" credential.

Overall, this pipeline script provides an efficient and automated way to build, test, analyze, and deploy a Spring Boot web application using the Jenkins continuous integration server.

This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD and Kubernetes.

Now push the repository to your GitHub. Before pushing to GitHub don't forget to change the required configurations like GitHub Repo details, Sonar URL, DockerHub details, 

```
cd Project3---java-maven-sonar-argocd-k8s
#delete the .git directory ex: rm -rf .git 
git init
git add .
git commit -m "commit-message"
git remote add origin <your-github-repo-url>
git push origin master
```

![Screenshot (228)](https://user-images.githubusercontent.com/129657174/230668342-f1cf926f-1611-4373-8543-39572f14c485.png)
   
#### Create a new Jenkins pipeline job:
- In Jenkins, create a new pipeline job and configure it with the Git repository URL for the Java application.

![Screenshot (203)](https://user-images.githubusercontent.com/129657174/230661251-65d35f14-03d7-40e5-9e67-e5b3876ea187.png)
![Screenshot (205)](https://user-images.githubusercontent.com/129657174/230661264-94106df5-ebc2-4362-b051-0ac7003716d2.png)
   
- Now Build the Job and wait for the build process to complete.
- Monitor the pipeline stages and fix any issues that arise.
   
![Screenshot (213)](https://user-images.githubusercontent.com/129657174/230661391-82561f42-1a29-443a-a086-fa1db0a6bdf5.png)

Once the build is successful, go to the sonarqube server and check the projects results.
   
![Screenshot (214)](https://user-images.githubusercontent.com/129657174/230661450-a165aa44-7a46-455c-bc85-3a12939a6909.png)
![Screenshot (215)](https://user-images.githubusercontent.com/129657174/230661457-7159d13f-73de-4594-b3df-cb94eec3544b.png)

Now go to Docker Hub Repository and check for the pushed image
   
![Screenshot (216)](https://user-images.githubusercontent.com/129657174/230661530-a1755187-5ff0-4fe7-b820-a2a2eac1442b.png)
   
Now go to Argo CD server, refresh the page and check deployments.
   
![Screenshot (219)](https://user-images.githubusercontent.com/129657174/230661604-5d9a3f69-b448-4d3e-b3f8-e0fbd3f4740f.png)

Now to access the server within the network

```
   vim spring-service.yml
```
   
![Screenshot (222)](https://user-images.githubusercontent.com/129657174/230662910-953362ff-4afb-4450-9b85-eb752069d1d2.png)

![Screenshot (224)](https://user-images.githubusercontent.com/129657174/230662961-0aa75dfd-a8f4-497f-8e8b-14707d7b7aae.png)

Now copy the service ip address in the browser
   
![Screenshot (225)](https://user-images.githubusercontent.com/129657174/230663088-012533e6-a048-42c3-b981-462a9b37ebf7.png)

### Hurrayyyyyyyyy! We have deployed the application successfully.

**Hope you all are Enjoyed**

[My Blog](https://iamsaikishore.hashnode.dev/spring-boot-based-java-web-application-using-maven-sonarqube-argo-cd-and-kubernetes)

For the original Project details click the below links.

[GitHub Repo](https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero)

[Youtube](https://www.youtube.com/watch?v=JGQI5pkK82w&t=1252s)


