# GKE STATIC ENDPOINT

Get comments by org-name

GET http://34.149.193.224/comment-management/orgs/{org-name}/comments

- response status code 200

- data comments array

```json
[
    {
        "id": 10,
        "comment": "like it is very good!",
        "deleteMark": false,
        "createdAt": "2021-09-20T20:09:09.392Z",
        "updatedAt": "2021-09-20T20:09:09.398Z",
        "organizationId": 1
    }
]
```

Add comment to organization

POST http://34.149.193.224/comment-management/orgs/{org-name}/comments

- response status code 204

Soft delete all comment for specific organization

DELETE http://34.149.193.224/comment-management/orgs/{org-name}/comments

- response status code 204

Get members by org-name. and show the member information

GET http://34.149.193.224/member-management/orgs/{org-name}/members

- response status code 200

```json
[
    {
        "id": 3,
        "userName": "zeddd",
        "avatarUrl": "avatarUrl",
        "followers": "2222",
        "following": "113333",
        "createdAt": "2021-09-20T19:00:22.467Z",
        "updatedAt": "2021-09-20T19:00:22.765Z",
        "organizationId": 1
    }
]
```

Create member

POST http://34.149.193.224/member-management/orgs/{org-name}/members

- response status code 204



# Google kubernetes engine architecture and docuemnt

### Prerequisites

- Google Cloud SDK

## Install gcloud

https://cloud.google.com/sdk/docs/install#mac
gcloud command not found - while installing Google Cloud SDK

## Gcloud project setup

```sh
# select project id xend-326306
# default zone us-west1-a
$ gcloud init

# If already have a account. Please logout first
$ gcloud auth revoke
```

# Google kubernetes engine architecture

## Create gke cluster

```sh
# Setup to the correct zone and project id
gcloud config set project xend-326306
gcloud config set compute/zone us-west1-a


# Create gke cluster
gcloud container clusters create loadbalance

# Show the created cluster
gcloud compute instances list | grep gke

```

## Setup deployment

Deployments represent a set of multiple, identical Pods with no unique identities. A Deployment runs multiple replicas of your application and automatically replaces any instances that fail or become unresponsive.

```sh
# Setup web deployment
$ gcloud apply -f deployment-web.yaml
# Setup web2 deployment
$ gcloud apply -f deployment-web2.yaml
```

## Setup service

The idea of a Service is to group a set of Pod endpoints into a single resource. You can configure various ways to access the grouping. By default, you get a stable cluster IP address that clients inside the cluster can use to contact Pods in the Service.

```sh
# Setup web service
$ gcloud apply -f service-web.yaml
# Setup web2 service
$ gcloud apply -f service-web2.yaml
```

## Setup ingress

In GKE, an Ingress object defines rules for routing HTTP(S) traffic to applications running in a cluster. An Ingress object is associated with one or more Service objects, each of which is associated with a set of Pods

```sh
# Setup ingess
$ gcloud apply -f ingress.yaml
```

## Setup secret

Secrets are secure objects which store sensitive data, such as passwords, OAuth tokens, and SSH keys in your clusters. Storing sensitive data in Secrets is more secure than in plaintext ConfigMaps or in Pod specifications. Using Secrets gives you control over how sensitive data is used, and reduces the risk of exposing the data to unauthorized users.

```sh
# Craete database secret
kubectl create secret generic db-secret \
  --from-literal=username=<USER-NAME> \
  --from-literal=password=<PASSWORD> \
  --from-literal=database=<DATABASE>


kubectl create secret generic db-connection-ip \
    --from-literal=db_host=<DATABASE-IP>
```

## Setup Static IP for ingress

```sh
gcloud compute addresses create web-static-ip --global
```

### setup static ip in ingress yaml

```yaml
metadata:
  name: basic-ingress
  annotations:
    kubernetes.io/ingress.global-static-ip-name: "web-static-ip"
```

### Use secret in gke deployment yaml file

<!-- For the container -->

```yaml
env:
  - name: DB_ACCOUNT
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: username
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
  - name: DATABASE
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: database
  - name: DB_HOST
    valueFrom:
      secretKeyRef:
        name: db-connection-ip
        key: db_host
```

# Swagger api document

### For comment management

```sh
git clone git@github.com:wgod58/comment-management.git

cd comment-management

yarn install

yarn run start

```

Open browser and check this endpoint

- http://localhost:5010/comment-management/api-docs/

### For comment management

```sh
git clone git@github.com:wgod58/comment-management.git

cd comment-management

yarn install

yarn run start

```

Open browser and check this endpoint

- http://localhost:5020/member-management/api-docs/

# The DB connection with localhost develop

The Cloud SQL Auth proxy provides secure access to your instances without the need for Authorized networks or for configuring SSL.

```sh
curl -o cloud_sql_proxy https://dl.google.com/cloudsql/cloud_sql_proxy.darwin.386

# Update access
chmod +x cloud_sql_proxy

# open proxy connection to the cloud sql
./cloud_sql_proxy -instances=xend-326306:us-west1:xendit=tcp:5432

```

### After open the proxy server

```
# Ask dev team to get the DB INFO

# edit .env file
DATABASE=postgres
DB_HOST=127.0.0.1
DB_PORT=5432
DB_ACCOUNT=
DB_PASSWORD=
```

### Deployment CI/CD

- ##### Google Cloud Build

  - **How do you update your docker image for deployment ?**

    - A cloud build trigger is set up for repo

      - when develop branch update, the `cloudbuild.yaml` script will be executed by Google Cloud Build automatically and a new docker image will be built and save inside `Container Registry`

      - Once the docker image is updated. Cloud Build will execute the remaining commands inside `cloudbuild.yaml` in which the latest docker image will be deployed to `Google Kubernetes Engine` as a running service.

      - the command in cloudbuild includes auto testing, push image and deploy image

      - To sum up, developers do not need to worry about deployment.

      - For more information about Cloud Build please view the Google GCloud Platform Documentation
