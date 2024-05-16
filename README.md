### Authors : BERTIN Samuel, BRETECHE Youenn
# Deployment Instructions

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Registry Setup](#registry-setup)
   - [Images](#images)
   - [Compose](#compose)
3. [Kubernetes (Stand-Alone)](#kubernetes-stand-alone)
4. [Kubernetes with Ansible (Hybrid)](#kubernetes-with-ansible-hybrid)
   - [Virtual Machines Setup](#virtual-machines-setup)
   - [Firewall Setup](#firewall-setup)
   - [Ansible](#ansible)
   - [Kubernetes](#kubernetes)
5. [Kubernetes with Ansible (Hybrid - Part 4)](#kubernetes-with-ansible-hybrid-part-4)
   - [Virtual Machines Setup](#virtual-machines-setup-1)
   - [Firewall Setup](#firewall-setup-1)
   - [Ansible](#ansible-1)
   - [Kubernetes](#kubernetes-1)

## Prerequisites <a name="prerequisites"></a>

You should have the following tools installed on your machine:

- [docker](https://docs.docker.com/engine/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Google Cloud CLI](https://cloud.google.com/sdk/docs/install)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#pipx-install) (only available on Linux and MacOS, mandatory for the Ansible part of the project)

Connect to your GCP account and link your computer with a new or existing GCP Project.

```
gcloud config set project PROJECT-ID
```

Once connected, make sure to enable Kubernetes Engine and connect to a new or existing cluster from GKE.

```
gcloud container clusters get-credentials <CLUSTER-ID> --zone europe-west9-a --project <PROJECT-ID>
```

## Registry Setup <a name="registry-setup"></a>

### Images <a name="images"></a>

For each service, we have associated a Docker image:

- seed: `./seed/Dockerfile`
- vote: `./vote/Dockerfile`
- worker: `./worker/Dockerfile`
- result: `./result/Dockerfile`
- nginx: `./nginx/Dockerfile`
- postgres: `./postgres/Dockerfile`
- redis: `./redis/Dockerfile`

These images must be present for the application to function. If you modify the images, ensure that you can `build` them without error from Docker. The postgres and redis images have been specifically modified to include the necessary scripts for healthcheck.

### Compose <a name="compose"></a>

We have made the use of services easier via Docker Compose. For the application to work with Kubernetes, you must first ensure that you push the images from the Docker Compose into a Docker registry.

If you do not know how to push your images to a Docker registry, you can follow the preliminary configuration proposed [here](https://gitlab.imt-atlantique.fr/login-nuage/project#preliminary-phase-push-your-docker-images-into-a-gcp-container-registry).

We will use the address of these images in our Kubernetes manifests.

> **Warning !**
> Make sure to replace the remote images address with the ones from your own registry.

## Kubernetes (Stand-Alone) <a name="kubernetes-stand-alone"></a>

Once the preliminary phase of making the images of the application available is complete, we can proceed to use Kubernetes.

**From the `./kube/kustomization` folder**, deploy your config:

```
kubectl apply -k .
```

**From the `./kube` folder**, deploy your resources:

```
kubectl apply -f .
```

If you want to change the autoscaling params:

```
kubectl autoscale deployment vote-hpa --cpu-percent=70 --min=2 --max=10
```

The job should complete in around 10 seconds. You can connect through vote and result services:

```
kubectl get svc

vote => http://<vote-public-ip>:8000
result => http://<result-public-ip>:3000
```

## Kubernetes with Ansible (Hybrid) <a name="kubernetes-with-ansible-hybrid"></a>

### Virtual Machines Setup <a name="virtual-machines-setup"></a>

The hybrid version is pretty similar to the stand-alone version of our application. It will be using **Ansible** to deploy the database configurations needed for our app.

First of all, you will need to instantiate a virtual machine within the same zone as your cluster/project. For this deployment purpose, we chose to work with E2 default machines on GCP. We recommend you to name your machine `postgres`.

Once you've set up the machines, you will need to tag them as such:

```
gcloud compute instances add-tags <INSTANCE-NAME> --tags=db-server
```

### Firewall Setup <a name="firewall-setup"></a>

Add a firewall rule on GCP to allow incoming connections on port 5432, with our tag as the target:

1. On the GCP dashboard, search for the "Firewall" service.
2. Click "Create a firewall rule." Choose a name for the rule.
3. In "Target tags," enter your tag `db-server`.
4. In "IPv4 ranges," enter `0.0.0.0/0` to allow all source IPs.
5. In "Protocols and ports," select "TCP" and enter the PostgreSQL port `5432`.
6. To verify that the rule is applied correctly, in the GCP "Firewall" service, click on the name of the rule to see the details. At the bottom of the page, a section indicates the resources to which it is applied; the VM should appear there. It may take a little while.

### Ansible <a name="ansible"></a>

Move to `/ansible/voting-app/inventories/gcp.yaml`
- Modify the `ansible_host` from `postgres` with your PrimaryVM public IP adress
- Modify the `ansible_user` as the user name as the one running the script
- Modify the `ansible_ssh_private_key_file` with the path to your private key file

From the `/ansible/voting-app` folder, launch the deployment playbook:

```
ansible-playbook deploy_postgres.yaml
```

### Kubernetes <a name="kubernetes"></a>

Make sure that you have done the prerequisites similarly to the Kubernetes part (connect project and cluster).

Move back to `/ansible`:

- In `db-endpoint.yaml` change the IP address for the public IP of your VM.

Launch the kubernetes manifests:

```
kubectl apply -f .
```

You can connect through vote and result services:

```
kubectl get svc

vote => http://<vote-public-ip>:8000
result => http://<result-public-ip>:3000
```

Once all the votes collected you can run a backup of your database using the appropriate Ansible playbook from `/ansible/voting-app`:

```
ansible-playbook dump_postgres.yaml
```

## Kubernetes with Ansible (Hybrid - Part 4) <a name="kubernetes-with-ansible-hybrid-part-4"></a>

### Virtual Machines Setup <a name="virtual-machines-setup-1"></a>

The hybrid version is pretty similar to the stand-alone version of our application. It will be using **Ansible** to deploy the database(s) configurations needed for our app.

First of all, you will need to instantiate 2 virtual machines within the same zone as your cluster/project. For this deployment purpose, we chose to work with E2 default machines on GCP. We recommend you to name your machines PrimaryVM and StandbyVM.

Once you've set up the machines, you will need to tag them as such:

```
gcloud compute instances add-tags <INSTANCE-NAME> --tags=db-server
```

### Firewall Setup <a name="firewall-setup-1"></a>

Add a firewall rule on GCP to allow incoming connections on port 5432, with our tag as the target:

1. On the GCP dashboard, search for the "Firewall" service.
2. Click "Create a firewall rule." Choose a name for the rule.
3. In "Target tags," enter your tag `db-server`.
4. In "IPv4 ranges," enter `0.0.0.0/0` to allow all source IPs.
5. In "Protocols and ports," select "TCP" and enter the PostgreSQL port `5432`.
6. To verify that the rule is applied correctly, in the GCP "Firewall" service, click on the name of the rule to see the details. At the bottom of the page, a section indicates the resources to which it is applied; the VM should appear there. It may take a little while.

### Ansible <a name="ansible-1"></a>

Move to `/ansible-part4/voting-app/inventories/gcp.yaml`
- Modify the `ansible_host` from `db1` with your PrimaryVM public IP adress
- Modify the `ansible_host` from `db2` with your StandbyVM public IP adress
- Modify the `ansible_user` as the user name as the one running the script
- Modify the `ansible_ssh_private_key_file` with the path to your private key file

From the `/ansible-part4/voting-app` folder, launch the deployment playbook:

```
ansible-playbook deploy_with_replication.yaml
```

### Kubernetes <a name="kubernetes-1"></a>

Make sure that you have done the prerequisites similarly to the Kubernetes part (connect project and cluster).

Move back to `/ansible-part4`:

- In `02-db-endpoint.yaml` change the IP address for the public IP of your PrimaryVM.

- In `02-db-replica-endpoint.yaml` change the IP address for the public IP of your StandbyVM.

Launch the kubernetes manifests:

```
kubectl apply -f .
```

You can connect through vote and result services:

default namespace

```
kubectl get svc

vote => http://<vote-public-ip>:8000
```

db-replica namespace

```
kubectl get svc -n db-replica

result => http://<result-public-ip>:3000
```

Even though the worker sends the data to the PrimaryVM, the replication mechanism does not seem to work at the moment. We left the results here as the cluster configuration allows us to read from the replica and to write into the primary.
