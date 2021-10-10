# EKS Cluster with WordPress Application Exposed through Istio

This repository hosts the codebase to: 

Create an EKS cluster using Terraform
Deploy a Wordpress Application and Expose the application through Istio


#### Prerequisites: 

Created IAM role to access AWS resources from EC2 instance through  Terraform.
Created an EC2 instance and assigned IAM role to provision AWS resources.

client tools: Terraform v1.0.8, kubectl v1.22.0

#### Detailed steps:

Clone this Repo
```
git clone https://github.com/alentmathew/democluster.git 
```
Change directory
```
cd democluster
```

Terraform initialise
```
terraform init 
```

Terraform plan to view all the changes to be performed
The cluster name and subnets are defined in terraform.tfvars file
```
terraform plan --var-file=terraform.tfvars
```

Terraform apply to execute the changes
```
terraform apply --var-file=terraform.tfvars
```
This will inititate EKS cluster creation. kube config is also set up so that the communication to apiserver is established.

![eks-cluster](https://github.com/alentmathew/democluster/blob/main/images/ekscluster.PNG)



See the status of nodes
```
kubectl get nodes
```

### Deployment of WordPress application:

Mysql password saved as secret

```
echo -n <password> | base64
```
```
kubectl apply -f ./yamls/secret.yaml
```

Deploying the MySQL database:
```
kubectl apply -f ./yamls/mysql-deployment.yaml
```

Checking for PersistentVolume
```
kubectl get pv
```

Checking for PersistentVolumeClaim
```
kubectl get pvc
```

Check the status of pods:
```
kubectl get pods
```

Check the status of service
```
kubectl get service
```

Deploying WordPress application
```
kubectl apply -f ./yamls/wordpress-deployment.yaml
```

Checking for PersistentVolume
```
kubectl get pv
```

Checking for PersistentVolumeClaim
```
kubectl get pvc
```
Check the status of pods:
```
kubectl get pods
```

Check the status of service:
```
kubectl get service
```
![pods](https://github.com/alentmathew/democluster/blob/main/images/podnoddeploy.PNG)


## Istio mesh configuration

![Istio](https://istio.io/latest/docs/ops/deployment/architecture/arch.svg)


Create istio-system namespace:

```
kubectl create namespace istio-system
```


Download Istio 
```
curl -L https://istio.io/downloadIstio | sh -
```

Move to the Istio package directory
```
cd istio-1.11.3
```

The installation directory contains:
1. Sample applications in samples/
2. The istioctl client binary in the bin/ directory.

Add the istioctl client to your path (Linux or macOS):
```
export PATH=$PWD/bin:$PATH
```

Install Istio using istioctl
```
istioctl install --set profile=demo -y
```
✔ Istio core installed
✔ Istiod installed
✔ Egress gateways installed
✔ Ingress gateways installed
✔ Installation complete

Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:
```
kubectl label namespace default istio-injection=enabled
```

Since Istio was installed after deploying Wordpress and MySQL, istio sidecar envoy is not attached. Therefore, we need to restart the pods.
```
kubectl delete po wordpress-5994d99f46-2282s
```
```
kubectl delete po wordpress-mysql-6c479567b-fb252
```

After Restart, the Pods will be equipped with Istio sidecar envoy so that traffic can be enabled.

Finally, to expose the App, associate the Application  with the Istio Gateway and VirtualService
```
kubectl apply -f ./yamls/istio-gateway-virtualservice.yaml
```

Ensure that there are no issues with the configuration:
```
istioctl analyze
```
✔ No validation issues found when analyzing namespace: default.


Get LoadBalancer hostname to access WordPress application 
```
kubectl get svc istio-ingressgateway -n istio-system
```

![istiolb](https://github.com/alentmathew/democluster/blob/main/images/istiolb.PNG)


Now the WordPress application is exposed through the Istio Mesh

![wordpress](https://github.com/alentmathew/democluster/blob/main/images/wp.PNG)


## Deletion:

Deleting Istio gateway and virtualservice
```
kubectl delete -f ./yamls/istio-gateway-virtualservice.yaml
```

deleting wordpress-deployment
```
kubectl delete -f ./yamls/wordpress-deployment.yaml
```

deleting MySQL db and secret

```
kubectl delete -f ./yamls/mysql-deployment.yaml
```
```
kubectl delete -f ./yamls/secret.yaml
```

Delete the cluster using Terraform destroy
```
terraform destroy --var-file=terrafrom.tfvars
```

