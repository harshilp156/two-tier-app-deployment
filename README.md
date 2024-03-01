
# TWO Tier app deployment on AWS EKS


**Architecture**

The architecture comprises a two-tier Flask application with a MySQL database backend. Docker containers are used to handle the application components, ensuring consistency and ease of deployment. Kubernetes orchestrates the containerized application, managing resources and scaling dynamically to meet demand. Deployment on AWS EKS improves the scalability and resilience of managed Kubernetes services, fault tolerance and enabling seamless operations.

**Key Tasks:**

1. Containerization with Docker:

    Used Docker and Docker Compose to containerize the Flask    application and MySQL database.
    
    Images were pushed to DockerHub repository for versioning and accessibility.

2. Kubernetes Cluster Setup:

    Automated Kubernetes cluster setup using kubeadm to deploy.

    Transitioned to AWS EKS with eksctl for improved management and scalability.

3. Helm Chart Deployment:

    Packaged Kubernetes Manifest files using Helm, streamlining deployment and management.

    Used Helm charts to deploy the application on AWS EKS efficiently.

4. High Availability and Scalability:

    Implemented a multi-node cluster setup to ensure high availability and fault tolerance.

    Used load balancer for distributing traffic, optimizing performance and scalability.

**Conclusion:**

Conclusion: The two-tier Flask application's successful deployment on AWS EKS shows the advantages of containerisation and Kubernetes orchestration for modern application deployments. This project improves fault tolerance and scalability while also highlighting the operational benefits of using managed cloud services, such as AWS EKS, for more reliable and efficient operations.


## Docker Installation

Here I have dockerized the application using AWS EC2 ubuntu instance(Free tier).

```bash
sudo apt update
sudo apt installdocker.io
```
Cloned the project from git into the EC2 instance.

DockerFile:
```bash
# Use an official Python runtime as the base image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# install required packages for system
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Copy the requirements file into the container
COPY requirements.txt .

# Install app dependencies
RUN pip install mysqlclient
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Specify the command to run your application
CMD ["python", "app.py"]
```

Now build the docker file.

```bash
docker build . -t flaskapp
```
Here I have created the docker network to connect these two docker containers.

```bash
docker network create twotier 
docker run -d -p 5000:5000 --network=twotier -e MYSQL_HOST=mysql -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_DB=mydb --name=flaskapp:latest
docker run -d -p 3306:3306 --network=twotier -e MYSQL_DATABASE=mydb -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_ROOT_PASSWORD=admin --name=mysql:5.7
docker ps
docker network ps
docker netowork inspect
```
Now Docker login to push the conatiner to docker hub.

```bash
docker login
docker tag flaskapp:latest harshilrpatel/flaskapp:latest
docker push harshilrpatel/flaskapp:latest
```

Now installed docker-compose and wrote docker-compose.yml file
```bash
sudo apt install docker-compose
```
```bash
version: '3'
services:
  backend:
    image:harshilrpatel/flaskapp:latest
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: admin
      MYSQL_PASSWORD: admin
      MYSQL_DB: myDb
    depends_on:
      - mysql

  mysql:
    image: mysql:5.7
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myDb
      MYSQL_USER: admin
      MYSQL_PASSWORD: admin
    volumes:
      - ./message.sql:/docker-entrypoint-initdb.d/message.sql   # Mount sql script into container's /docker-entrypoint-initdb.d directory to get table automatically created
      - mysql-data:/var/lib/mysql  # Mount the volume for MySQL data storage

volumes:
  mysql-data:
```

```bash
docker-compose up -d
```
![Screenshot (556)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/ef1f5c42-1f40-42e6-94c5-74ba7757806b)

## EKS Setup

Here I have done the setup of AWS EKS using AWS EC2 instance (freetier).

![Screenshot (558)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/9f186fc5-d0a0-4124-84e0-410a9671a8fe)

![Screenshot (559)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/01565962-ac95-4223-8b40-5e576b1fce64)

![Screenshot (560)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/1a9b2718-8e65-4553-a825-7b04cc84d446)

![Screenshot (561)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/0ab374b5-f857-418b-b4e1-3de1c5000770)

![Screenshot (562)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/460d5672-c06b-4262-8884-89900bb62c77)


Installed AWS CLI on that instance.

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o
"awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
```

Created an IAM role with the necessary permissions for EKS and attached it to an IAM User.

After creating the user, generated the Access Key and Secret Access
Key for this user.

Configured AWS CLI.

Installed Kubectl :
```bash
curl -o kubectl https://amazon-eks.s3.us-west2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short –client
```

Installed eksctl :

```bash
curl --silent --location
"https://github.com/weaveworks/eksctl/releases/latest/download/
eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

EKS Cluster Setup :

```bash
eksctl create cluster --name <cluster-name> --region <region> --node-type
t2.micro --nodes-min 2 --nodes-max 2
```
Now clone the git repository into the cluster.










## Helm Chart

Here I have created helm charts for flask app and mysql database.

**Creating mysql-chart**
Commands:

```bash
helm create mysql-chart
```
Made the necessary changes to the Values.yaml and templates/deployment.yaml files.

- Key changes to the values.yaml file.

    1. Adding mysql latest image.
    2. Changing mysql port to 3306.
    3. Adding mysql credentials environment values.

- Key changes to the templates/deployment.yaml file.

    1. Adding Mysql credentials environment. 

```bash
helm package mysql-chart
helm install mysql-chart ./mysql-chart
```

**Creating flask-app-chart**

Commands:

```bash
helm create flask-app-chart
```
Made the necessary changes to the Values.yaml and templates/deployment.yaml files.

- Key changes to the values.yaml file.

    1. Adding flask app latest image docker-hub.
    2. Changing flask-app ports.


    service:

        type: LoadBalancer
        port: 80
        targetPort: 5000


    3. Adding mysql credentials environment values.
    4. Changes to templates/deployment.yaml file.


- Key changes to the templates/deployment.yaml file.

    1. Adding Mysql credentials environment. 

```bash
helm package flask-app-chart
helm install flask-app-chart ./flask-app-chart
```

Now we can access the website using below commands:

```bash
kubectl get all
kubectl get svc
```

The website is accessible using flask-app-chart svc.
![Screenshot (557)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/f1fb1361-5a54-40d4-bb32-ee28093a93f1)
![Screenshot (563)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/acdfbb09-78bf-412a-aa2e-aa1f37bfb451)
![Screenshot (564)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/e04a830b-7e29-423b-a7a6-8ffbc0d4b5f6)
![Screenshot (565)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/40b11314-e38e-45c4-a9ee-08e21f77e2ba)
![Screenshot (566)](https://github.com/harshilp156/two-tier-app-deployment/assets/67538347/b3efd2f3-50fe-4236-8189-cc4c18b0c382)

## Tech Stack

**AWS:** EKS, IAM, EC2

**Docker**

**Helm**

**Kubernetes**

**Flask** **HTML** **CSS** **VIM**

**GIT**
