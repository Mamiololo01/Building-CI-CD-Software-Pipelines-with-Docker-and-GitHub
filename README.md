# Building-CI-CD-Software-Pipelines-with-Docker-and-GitHub
Building CI/CD Software Pipelines with Docker and GitHub

Docker has revolutionized the way DevOps teams approach software development and deployment.

By allowing applications to run within isolated and lightweight containers, Docker makes it easier to develop, test, and deploy software in a variety of environments.

DevOps teams can use Docker to standardize their development and deployment processes, reducing the risk of errors and inconsistencies.

Docker also allows teams to scale applications more efficiently by running multiple containers on a single host.

Suppose you work for a software development company that provides cloud-based applications to clients. The company has a development team that creates and maintains the software applications and a separate operations team that manages the cloud infrastructure and deployment process.

The development team uses GitHub to manage the application code written using Boto3 in Python in order to interact with various cloud services (e.g. AWS S3, EC2, etc.).

Traditionally, the development team would manually deploy the applications to the cloud infrastructure managed by the operations team, which often lead to errors and delays.

To streamline the deployment process and improve collaboration between the development and operations teams, your company can implement a containerized approach using Docker.

In this article, I will show you how. But before we dive in, I am going to assume that you have some previous experience:

Pre-requisites

Proficiency in Docker and containerization concepts

Experience creating Docker images and managing containers

Knowledge Git & GitHub version control systems

Network security

Proficiency with the AWS cloud console and services


Step 1: Create our File System & Cloning our GitHub Repos

Per our use-case, we need to build three different Dockerfiles. We are going to build ours from an Ubuntu image.

First, in Cloud9, we’re going to create a directory for our Docker Files and navigate into our newly made directory

mkdir dockerfiles

cd dockerfiles

Next, we’re going to clone our three GitHub repositories into separate subdirectories within this directory.

git clone https://github.com/hogtai/Python_Scripts repo1

git clone https://github.com/hogtai/AWS_Scripts repo2

git clone https://github.com/hogtai/xrpl-dev-portal repo3

Step 2: Creating our 3 Docker Files

Next, we need to create a Dockerfile for each repository, specifying the base image as ubuntu.

Each Docker file will also install python / boto3 since our containers will need to run python scripts that need to interact with other AWS services and must be specific to a particular repo. One repo per container.

Here is our Dockerfile for our “Development” container:

FROM ubuntu

RUN apt-get update && apt-get install -y git python3-pip

RUN git clone <repo1-url #1>

WORKDIR /<repo1-dir #1>

RUN pip3 install -r requirements.txt

CMD ["python3", "app.py"]

Here is our Dockerfile for our first “Production” container:

FROM ubuntu

RUN apt-get update && apt-get install -y git python3-pip

RUN git clone <repo1-url #2>

WORKDIR /<repo1-dir #2>

RUN pip3 install -r requirements.txt

CMD ["python3", "app.py"]

Here is our Dockerfile for our second “Production” container:

FROM ubuntu

RUN apt-get update && apt-get install -y git python3-pip

RUN git clone <repo1-url #3>

WORKDIR /<repo1-dir #3>

RUN pip3 install -r requirements.txt

CMD ["python3", "app.py"]


Step 2: Building our Docker Images

Once our Dockerfiles are defined, we need to build them into three separate images because of the three different GitHub repos that they will be pull files from

docker build -t dockerdev -f ~/environment/dockerfiles/repo1/dockerdev

docker build -t dockerprod1 -f ~/environment/dockerfiles/repo2/dockerprod1

docker build -t dockerprod2 -f ~/environment/dockerfiles/repo3/dockerprod2

These commands need to be run with the full path specified. Once the Dockerfiles have been successfully built, we will receive confirmation in our console:

dockerdev image was successfully built

dockerprod1 image was successfully built

dockerprod2 image successfully built

Finally, we can use the docker images command to list all of our images on our local host


Step 3: Building our Docker Containers with Bind Mounts

In order to download the repositories to the containers we use the following command to create three separate containers from our three images.

We use the flag -d to run the containers detached and add the sleep command because our containers are not running any active scripts and during set up we want them to be running to complete our pipeline.

docker run -it --name dev -d $(pwd)/src:/app dockerdev sleep 100000
docker run -it --name prod1 -d $(pwd)/src:/app dockerprod1 sleep 100000
docker run -it --name prod2 -d $(pwd)/src:/app dockerprod1 sleep 100000

We have successfully created our containers

Bind mounts were deployed successfully

Now that our containers are running, we use the following command to copy the contents of each repository into it’s corresponding container:

docker cp ~/environment/dockerfiles/repo1 dev:/dev

docker cp ~/environment/dockerfiles/repo2 prod1:/prod

docker cp ~/environment/dockerfiles/repo3 prod2:/prod

We can verify our results with the following commands

docker exec dev ls /dev

docker exec prod1 ls /prod

docker exec prod2 ls /prod

Our repos were succesfully copied to their respective containers

Step 4: Defining & Configuring our Docker Networks

Revisiting our use-case, container one should be placed on a network called “Development” while containers two & three should be placed on a network called “Production”.

To create these networks and add the containers to them, we start by using the following command to create the “Development” network and add our ‘dev’ container to it.

By adding only our development containers to this network.

This will isolate this container from any network communication attempts from other containers in it’s environment.

docker network create development

docker network connect development dev

docker network ls

We have confirmation the our “Development” network was created, see that our ‘dev’ container was placed in that network and that we have configured our network to spec

Then, we use the following command to create the production network and add containers two and three to it:

docker network create production

docker network connect production prod1

docker network connect production prod2

docker network inspect production

We have confirmation the our “Production” network was created, see that our ‘prod1’ & ‘prod2’ containers were placed in that network and that we have configured our production network was configured to spec

Step 5: Testing our Container Networks

To verify that the “Development” container cannot communicate with the “Production” containers, we start by using the following command to log into the “Development” container:

docker exec -it dev bash
Once we’ve logged into the container. We need to install the “ping” package”:

apt-get install -y iputils-ping

Once the package is downloaded and installed, we use the following command to attempt to ping both our prod1 & prod2 containers.

We should receive a “Name or service not known” error because we have isolated our dev environment containers from our prod containers.

ping PROD1

ping PROD2

We also need to ensure that both production containers can talk to each other. We repeat this process by shelling into one of our production containers, installing the same package, and running a ping command:

docker exec -it prod1 bash

apt-get install -y iputils-ping

ping PROD2





