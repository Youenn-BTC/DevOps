# Ansible Part 4 - README

## Kubernetes with Ansible (TP - PARTIE 4)

### Virtual Machines set up

The hybrid version is pretty similar to the stand-alone version of our application.

It will be using **Ansible** to deploy some of the configurations needed for our app.

First of all you will need to instantiate 2 virtual machines within the same zone as your cluster/project.

For this deployement purpose we chose to work with E2 default machines on GCP.

We recommend you to name your machines PrimaryVM and StandbyVM.

Once you've set up the machines you will need to tag them as such

```
gcloud compute instances add-tags <INSTANCE-NAME> --tags=db-server
```

### Firewall set up

Add a firewall rule on GCP to allow incoming connections on port 5432, with our tag as the target.

1. On the GCP dashboard, search for the "Firewall" service.
2. Click "Create a firewall rule." Choose a name for the rule.
3. In "Target tags," enter your tag `db-server`.
4. In "IPv4 ranges," enter `0.0.0.0/0` to allow all source IPs.
5. In "Protocols and ports," select "TCP" and enter the PostgreSQL port `5432`.
6. To verify that the rule is applied correctly, in the GCP "Firewall" service, click on the name of the rule to see the details. At the bottom of the page, a section indicates the resources to which it is applied; the VM should appear there. It may take a little while.

### Ansible

Move to `/ansible/voting-app/inventories/gcp.yaml` and modify the IP adress from `db1` with your PrimaryVM public adsress and `db2` from your StandbyVM public adress.

From the `/ansible/voting-app` folder, launch the deployement playbook :

```
ansible-playbook deploy_with_replication.yaml
```

### Kubernetes

Make sure that you have done the prerequisites similary to the Kubernetes part (connect project and cluster).

Move back to `/ansible`

- In `02-db-endoint.yaml` change the IP adress for the public IP of your PrimaryVM.

- In `02-db-replica-endoint.yaml` change the IP adress for the public IP of your StandbyVM.

Launch the kubernetes manifests

```
kubectl apply -f .
```

You can connect trough vote and result services

default namespace

```
kubectl get svc

vote => htpp://<vote-public-ip>:8000
```

db-replica namespace

```
kubectl get svc -n db-replica

result => htpp://<result-public-ip>:3000
```

Even thought the worker sends the data to the PrimaryVM the replication mecanism does not seem to work at the moment.
We left the results here as the cluster configuration allows us to read from the replica and to write into the primary.
