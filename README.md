# Use Terraform to create a managed kubernet cluster on gcp (GKE cluster)


The origin doc can be found [here](https://learn.hashicorp.com/tutorials/terraform/gke?in=terraform/kubernetes)

This sample repo also creates a VPC and subnet for the GKE cluster. This is not required but highly recommended to keep your GKE cluster isolated.


## Step 1. Install gcloud sdk, terraform, kubectl

## Step 2. Set up and initialize your Terraform workspace

```
git clone https://github.com/hashicorp/learn-terraform-provision-gke-cluster

cd learn-terraform-provision-gke-cluster
```

In the folder, you will find four files used to provision a VPC, subnets and a GKE cluster.

- **vpc.tf** provisions a VPC and subnet. A new VPC is created for this tutorial so it doesn't impact your existing cloud environment and resources. This file outputs region.

- **gke.tf** provisions a GKE cluster and a separately managed node pool (recommended). Separately managed node pools allows you to customize your Kubernetes cluster profile — this is useful if some Pods require more resources than others. You can learn more here. The number of nodes in the node pool is defined also defined here.

- **terraform.tfvars** is a template for the project_id and region variables.

- **versions.tf** sets the Terraform version to at least 0.14.


### 2.1 Update your terraform.tfvars file

Replace the values in your **terraform.tfvars** file with your **project_id** and **region**. Terraform will use these values to target your project when provisioning your resources. Your terraform.tfvars file should look like the following.

**terraform.tfvars**

```
project_id = "REPLACE_ME"
region     = "us-central1"
```

You can find the project your gcloud is configured to with this command.

```
$ gcloud config get-value project
```

The region has been defaulted to us-central1; you can change it to any region.


### 2.2 set up and run

```
terraform init

terraform apply
```

If you have the below error when you run the terraform. You need to setup gcloud default creds. 

```
Error: Attempted to load application default credentials since neither `credentials` nor `access_token` was 
set in the provider block.  No credentials loaded. To use your gcloud credentials, run 'gcloud auth application-default login'.  Original error: google: could not find default credentials. See
https://developers.google.com/accounts/docs/application-default-credentials for more information.
```

create gcloud default creds
```
gcloud auth application-default login
```

After the terraform finish the installation, we should see the below outputs

```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

kubernetes_cluster_host = "34.122.195.202"
kubernetes_cluster_name = "projet-insee-poc-gke"
project_id = "projet-insee-poc"
region = "us-central1"
```

## Step 3. Configure kubectl

In step2, you've provisioned your GKE cluster, you need to configure kubectl.

Run the following command to **retrieve the access credentials** for your cluster and **automatically configure kubectl**.

```
gcloud container clusters get-credentials $(terraform output -raw kubernetes_cluster_name) --region $(terraform output -raw region)
```

**You may see the following warning message when you try to retrieve your cluster credentials. This may be because your Kubernetes cluster is still initializing/updating. If this happens, you can still proceed to the next step.**

## Step 4. Deploy and access k8s dashboard

To verify your cluster is correctly configured and running, you will deploy the Kubernetes dashboard and navigate to it in your local browser.

While you can deploy the Kubernetes dashboard using Terraform, kubectl is used in this tutorial so you don't need to configure your Terraform Kubernetes Provider.

The following command will schedule the resources necessary for the dashboard.

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

Now, create a proxy server that will allow you to navigate to the dashboard from the browser on your local machine. This will continue running until you stop the process by pressing CTRL + C.

```
kubectl proxy
```

click [here](http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/) to access the k8s dashboard

## Step 5. Create token for accessing k8s dashboard

To use the Kubernetes dashboard, you need to create a **ClusterRoleBinding** and provide an **authorization token**. This gives the **cluster-admin permission to access the kubernetes-dashboard**. Authenticating using kubeconfig is not an option. You can read more about it in the Kubernetes documentation.

**Run below command in another terminal (do not close the kubectl proxy process) to create the ClusterRoleBinding resource.**

```
kubectl apply -f https://raw.githubusercontent.com/hashicorp/learn-terraform-provision-gke-cluster/master/kubernetes-dashboard-admin.rbac.yaml
```

Now we have the role binding, we can create the authorization token now.

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep service-controller-token | awk '{print $1}')
```

Copy the token of the output to the web UI. Now you should have access of the web UI. 


## Step 6. GKE nodes and node pool

On the Dashboard UI, click Nodes on the left hand menu.

Notice there are 6 nodes in your cluster, even though gke_num_nodes in your gke.tf file was set to 2. This is because a node pool was provisioned in each of the three zones within the region to provide high availability.


```
gcloud container clusters describe projet-insee-poc-gke --region us-central1 --format='default(locations)'
```

Replace **projet-insee-poc-gke** with the **kubernetes_cluster_name** value from your Terraform output.



