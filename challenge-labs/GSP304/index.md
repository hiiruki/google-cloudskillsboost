# [GSP304] Build and Deploy a Docker Image to a Kubernetes Cluster

### [GSP304](https://www.cloudskillsboost.google/focuses/1738?parent=catalog)

![Lab Banner](https://cdn.qwiklabs.com/GMOHykaqmlTHiqEeQXTySaMXYPHeIvaqa2qHEzw6Occ%3D)

---

Time: 1 hour 15 minutes<br>
Difficulty: Intermediate<br>
Price: 5 Credits

Quest: [Cloud Architecture: Design, Implement, and Manage](https://www.cloudskillsboost.google/quests/124)<br>

Last updated: May 25, 2023

---

## Challenge scenario

Your development team is interested in adopting a containerized microservices approach to application architecture. You need to test a sample application they have provided for you to make sure that that it can be deployed to a Google Kubernetes container. The development group provided a simple Go application called `echo-web` with a Dockerfile and the associated context that allows you to build a Docker image immediately.

## Your challenge

To test the deployment, you need to download the sample application, then build the Docker container image using a tag that allows it to be stored on the Container Registry. Once the image has been built, you'll push it out to the Container Registry before you can deploy it.

With the image prepared you can then create a Kubernetes cluster, then deploy the sample application to the cluster.

1. An application image with a v1 tag has been pushed to the gcr.io repository

    ```bash
    mkdir echo-web
    cd echo-web
    gsutil cp -r gs://$DEVSHELL_PROJECT_ID/echo-web.tar.gz .
    tar -xzf echo-web.tar.gz
    rm echo-web.tar.gz
    cd echo-web
    docker build -t echo-app:v1 .
    docker tag echo-app:v1 gcr.io/$DEVSHELL_PROJECT_ID/echo-app:v1
    docker push gcr.io/$DEVSHELL_PROJECT_ID/echo-app:v1
    ```

2. A new Kubernetes cluster exists (zone: us-central1-a)

    ```bash
    gcloud config set compute/zone us-central1-a

    gcloud container clusters create echo-cluster --num-nodes=2 --machine-type=n1-standard-2
    ```

3. Check that an application has been deployed to the cluster

    ```bash
    kubectl create deployment echo-web --image=gcr.io/$DEVSHELL_PROJECT_ID/echo-app:v1
    ```

4. Test that a service exists that responds to requests like Echo-app

    ```bash
    kubectl expose deployment echo-web --type=LoadBalancer --port 80 --target-port 8000
    ```

## Congratulations!

![Congratulations Badge](https://cdn.qwiklabs.com/GOodosAwxciMN42hNV4ZqZIwQ5eXORJcUSvZ2SAuXYI%3D)
