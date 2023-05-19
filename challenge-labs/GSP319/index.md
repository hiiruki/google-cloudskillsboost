# [GSP319] Build a Website on Google Cloud: Challenge Lab


### [GSP319](https://www.cloudskillsboost.google/focuses/11765?parent=catalog)

![](https://cdn.qwiklabs.com/GMOHykaqmlTHiqEeQXTySaMXYPHeIvaqa2qHEzw6Occ%3D)

---

Time: 1 hour 30 minutes<br>
Difficulty: Intermediate<br>
Price: 5 Credits

Quest: [Build a Website on Google Cloud](https://www.cloudskillsboost.google/quests/115)<br>

Last updated: May 19, 2023

---

## Challenge lab scenario

You have just started a new role at FancyStore, Inc.

Your task is to take the company's existing monolithic e-commerce website and break it into a series of logically separated microservices. The existing monolith code is sitting in a GitHub repo, and you will be expected to containerize this app and then refactor it.

You are expected to have the skills and knowledge for these tasks, so don't expect step-by-step guides.

You have been asked to take the lead on this, after the last team suffered from monolith-related burnout and left for greener pastures (literally, they are running a lavender farm now). You will be tasked with pulling down the source code, building a container from it (one of the farmers left you a Dockerfile), and then pushing it out to GKE.

You should first build, deploy, and test the Monolith, just to make sure that the source code is sound. After that, you should break out the constituent services into their own microservice deployments.

Some FancyStore, Inc. standards you should follow:

- Create your cluster in `us-central1`.
- Naming is normally *team-resource*, e.g. an instance could be named **fancystore-orderservice1**.
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination.
- Use the `n1-standard-1` machine type unless directed otherwise.


## Your challenge

As soon as you sit down at your desk and open your new laptop, you receive the following request to complete these tasks. Good luck!

## Setup

Export the following variables in the Cloud Shell:

```bash
export MONOLITH_IDENTIFIER=
export CLUSTER_NAME=
export ORDERS_IDENTIFIER=
export PRODUCTS_IDENTIFIER=
```

from the labs variables, you can copy the value of each variable and paste it in the cloud shell

![labs variable](./images/labs%20variable.png)

> **Note:** Don't forget to replace the value of each variable with the value of the labs variable

like this
![export variable](./images/export%20variable.png)

> **Note:** Don't forget to enable the API's

```bash
gcloud services enable cloudbuild.googleapis.com
gcloud services enable container.googleapis.com
```

## Task 1: Download the monolith code and build your container

First things first, you'll need to [clone your team's git repo](https://github.com/googlecodelabs/monolith-to-microservices.git). 

```bash
git clone https://github.com/googlecodelabs/monolith-to-microservices.git
```

There's a `setup.sh` script in the root directory of the project that you'll need to run to get your monolith container built up.

``` bash
cd ~/monolith-to-microservices

./setup.sh
```

After running the setup.sh script, ensure your Cloud Shell is running its latest version of nodeJS.

```bash
nvm install --lts
```

There's a Dockerfile located in the `~/monotlith-to-microservices/monolith` folder which you can use to build the application container. Before building the Docker container, you can preview the monolith application on **port 8080**.

> Note: You can skip previewing the application if you want to, but it's a good idea to make sure it's working before you containerize it.

```bash
cd ~/monolith-to-microservices/monolith

npm start
```

`CTRL+C` to stop the application.

You will have to run Cloud Build (in that monolith folder) to build it, then push it up to GCR. Name your artifact as follows:

- GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
- Image name: `MONOLITH_IDENTIFIER`
- Image version: 1.0.0

```bash
gcloud services enable cloudbuild.googleapis.com

gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/${MONOLITH_IDENTIFIER}:1.0.0 .
```

## Task 2: Create a kubernetes cluster and deploy the application

Create your cluster as follows:

- Cluster name: `CLUSTER_NAME`
- Region: us-central1-a
- Node count: 3

```bash
gcloud config set compute/zone us-central1-a

gcloud services enable container.googleapis.com

gcloud container clusters create $CLUSTER_NAME --num-nodes 3

gcloud container clusters get-credentials $CLUSTER_NAME
```

Create and expose your deployment as follows:

- Cluster name: `CLUSTER_NAME`
- Container name: `MONOLITH_IDENTIFIER`
- Container version: 1.0.0
- Application port: 8080
- Externally accessible port: 80

```bash
kubectl create deployment $MONOLITH_IDENTIFIER --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/${MONOLITH_IDENTIFIER}:1.0.0

kubectl expose deployment $MONOLITH_IDENTIFIER --type=LoadBalancer --port 80 --target-port 8080
```

Make note of the IP address that is assigned in the expose deployment operation. Use this command to get the IP address:

```bash
kubectl get service
```

![kubectl get service](./images/kubectl%20get%20services.png)

You should now be able to visit this IP address from your browser and see the following:

![fancy store](./images/fancy%20store.png)

## Task 3. Create new microservices

Below is the set of services which need to be containerized. Navigate to the source roots mentioned below, and upload the artifacts which are created to the Google Container Registry with the metadata indicated. Name your artifact as follows:

**Orders Microservice**

- Service root folder: `~/monolith-to-microservices/microservices/src/orders`
- GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
- Image name: `ORDERS_IDENTIFIER`
- Image version: 1.0.0

```bash
cd ~/monolith-to-microservices/microservices/src/orders

gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/${ORDERS_IDENTIFIER}:1.0.0 .
```

**Products Microservice**

- Service root folder: `~/monolith-to-microservices/microservices/src/products`
- GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
- Image name: `PRODUCTS_IDENTIFIER`
- Image version: 1.0.0

```bash
cd ~/monolith-to-microservices/microservices/src/products

gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/${PRODUCTS_IDENTIFIER}:1.0.0 .
```


## Task 4: Deploy the new microservices

Deploy these new containers following the same process that you followed for the `MONOLITH_IDENTIFIER` monolith. Note that these services will be listening on different ports, so make note of the port mappings in the table below. Create and expose your deployments as follows:

**Orders Microservice**

- Cluster name: `CLUSTER_NAME`
- Container name: `ORDERS_IDENTIFIER`
- Container version: 1.0.0
- Application port: 8081
- Externally accessible port: 80

```bash
kubectl create deployment $ORDERS_IDENTIFIER --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/${ORDERS_IDENTIFIER}:1.0.0

kubectl expose deployment $ORDERS_IDENTIFIER --type=LoadBalancer --port 80 --target-port 8081
```

**Products Microservice**

- Cluster name: `CLUSTER_NAME`
- Container name: `PRODUCTS_IDENTIFIER`
- Container version: 1.0.0
- Application port: 8082
- Externally accessible port: 80

```bash
kubectl create deployment $PRODUCTS_IDENTIFIER --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/${PRODUCTS_IDENTIFIER}:1.0.0

kubectl expose deployment $PRODUCTS_IDENTIFIER --type=LoadBalancer --port 80 --target-port 8082
```

Get the external IP addresses for the Orders and Products microservices:

```bash
kubectl get svc -w
```

`CTRL+C` to stop the command.

Now you can verify that the deployments were successful and that the services have been exposed by going to the following URLs in your browser:

- `http://ORDERS_EXTERNAL_IP/api/orders`
- `http://PRODUCTS_EXTERNAL_IP/api/products`

Write down the IP addresses for the Orders and Products microservices. You will need them in the next task.

## Task 5. Configure and deploy the Frontend microservice

>**Note**: You can use the lab method or use my method. **Choose one that suits you**.

1. My method (Using [sed](https://linux.die.net/man/1/sed) (stream editor) and using one-line command)

```bash
export ORDERS_SERVICE_IP=$(kubectl get services -o jsonpath="{.items[1].status.loadBalancer.ingress[0].ip}")

export PRODUCTS_SERVICE_IP=$(kubectl get services -o jsonpath="{.items[2].status.loadBalancer.ingress[0].ip}")
```

```bash
cd ~/monolith-to-microservices/react-app
sed -i "s/localhost:8081/$ORDERS_SERVICE_IP/g" .env
sed -i "s/localhost:8082/$PRODUCTS_SERVICE_IP/g" .env
npm run build
```

2. The lab method (Using [nano](https://linux.die.net/man/1/nano) text editor)

Use the `nano` editor to replace the local URL with the IP address of the new Products microservices.

```bash
cd ~/monolith-to-microservices/react-app
nano .env
```

When the editor opens, your file should look like this.

```bash
REACT_APP_ORDERS_URL=http://localhost:8081/api/orders
REACT_APP_PRODUCTS_URL=http://localhost:8082/api/products
```

Replace the `REACT_APP_ORDERS_URL` and `REACT_APP_PRODUCTS_URL` to the new format while replacing with your Orders and Product microservice IP addresses so it matches below.

```bash
REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders
REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products
```

Press **CTRL+O**, press **ENTER**, then **CTRL+X** to save the file in the `nano` editor. Now rebuild the frontend app before containerizing it.

```bash
npm run build
```

## Task 6: Create a containerized version of the Frontend microservice

The final step is to containerize and deploy the Frontend. Use Cloud Build to package up the contents of the Frontend service and push it up to the Google Container Registry.

- Service root folder: `~/monolith-to-microservices/microservices/src/frontend`
- GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
- Image name: `FRONTEND_IDENTIFIER`
- Image version: 1.0.0

```bash
cd ~/monolith-to-microservices/microservices/src/frontend

gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/${FRONTEND_IDENTIFIER}:1.0.0 .
```

## Task 7: Deploy the Frontend microservice

Deploy this container following the same process that you followed for the **Orders** and **Products** microservices. Create and expose your deployment as follows:

- Cluster name: `CLUSTER_NAME`
- Container name: `FRONTEND_IDENTIFIER`
- Container version: 1.0.0
- Application port: 8080
- Externally accessible port: 80

```bash
kubectl create deployment $FRONTEND_IDENTIFIER --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/${FRONTEND_IDENTIFIER}:1.0.0

kubectl expose deployment $FRONTEND_IDENTIFIER --type=LoadBalancer --port 80 --target-port 8080
```

```bash
kubectl get svc -w
```

`CTRL+C` to stop the command.

![kubectl get svc](./images/kubectl%20get%20svc.png)

Wait until you see the external IP address and check the progress.

## Congratulations!

![](https://cdn.qwiklabs.com/tDSBmZi3kH7QdPue8oTiKmR0kVc3UTudveGazkCgmxw%3D)
