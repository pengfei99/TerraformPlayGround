# install ingress-nginx to the k8s cluster

The origin doc is [here](https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke)


# Get cluster admin rights for your kubctl client

Initialize your user as a cluster-admin with the following command:


```
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)
```


# Install ingress

Note there are danger note in the doc. You need to setup firewall rule. But I did nothing

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.3/deploy/static/provider/cloud/deploy.yaml
```
