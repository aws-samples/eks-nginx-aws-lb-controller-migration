#  Migrating from NGINX Ingress Controller to AWS Load Balancer Controller

This project demonstrates how to migrate from NGINX Ingress Controller to AWS Load Balancer Controller. One of the key motivations for migration is that NGINX ingress controller provisions Classic Load Balancer(CLB) on AWS. As CLB does not support integration with AWS Web Application Firewall (AWS WAF), customers could migrate to Application Load Balancer (ALB) as an alternative. ALB supports integration with AWS WAF. 


The project is structured as follows

    .

    ├── 1-infrastructure --> Terraform code which creates 1. EKS cluster with necessary add-ons 2. Deploys NGINX Ingress Controller 3. Deploys AWS Application Load Balancer Controller
    ├── 2-sample-app-2048 --> Deploys a web based game 2048. This folder also contains ingress.yamls files for both nginx as well as aws application load balancer controller.
    ├── supporting modules
     

## Solution Overview 

![plot](./docs/nginx-awslb-migration/Architecture.png)


## Prerequisites :

1. Install AWS cli https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
2. Install kubectl https://kubernetes.io/docs/tasks/tools/
3. Install helm https://helm.sh/docs/intro/install/
4. Install terraform https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli



## Getting Started - Please follow these instructions in sequence

#### STEP 1 : Create required Infrastructure using terraform (Create EKS Cluster + Deploy NGINX Ingress Controller + Deploy AWS Load Balancer Controller )

```
cd 1-Infrastructure  
```
```
terraform init
```
```
terraform apply 
```

##### Infrastructure creation will take about 20 minutes.

#####  Output should look like below  

```
Apply complete! Resources: 66 added, 0 changed, 0 destroyed.

Outputs:

configure_kubectl = <<EOT
configure kubectl: aws eks --region us-east-1 update-kubeconfig --name nginx-awslb-controller 
```

##### Configure access to the cluster
```
aws eks --region us-east-1 update-kubeconfig --name nginx-awslb-controller 
```


#### STEP 2: Deploy Application ( 2048 game). This will deploy application pods, services and 2 ingress objects , one for nginx ingress controller and other ingress object for AWS Application Load Balancer Controller
    
```
cd ../2-sample-app-2048  

kubectl apply -f ./
```

#### Output should look like below
```
namespace/game-2048 created
deployment.apps/deployment-2048 created
service/service-2048 created
ingress.networking.k8s.io/alb-ingress created
ingress.networking.k8s.io/nginx-ingress created

```

### STEP 3: Application game-2048 can now be accessed from 2 paths 1/ via Classic Loadbalancer url + NGINX  2/ via Application Load Balancer url ( see the Solution Overview diagram above)

run the following command to get the CLB and ALB urls. Access them from the browser http://<ADDRESS>

```
 kubectl get ingress -A
```

#### Output should look like below

```
NAMESPACE   NAME            CLASS   HOSTS   ADDRESS                                                                   PORTS   AGE
game-2048   alb-ingress     alb     *       k8s-game2048-albingre-31e3d3a41a-91512281.us-east-1.elb.amazonaws.com     80      56s
game-2048   nginx-ingress   nginx   *       ae56424309b314a9ca3c89196258db1a-1610871911.us-east-1.elb.amazonaws.com   80      98m
```


### STEP 4: Manage migration from CLB to ALB url using DNS entries. Use DNS routing Policy (such as Weighted routing) to  incrementally shift traffic from CLB to ALB.


### STEP 5 : Destroy

To teardown and remove the resources created in this example:

```sh
cd ../1-Infrastructure 

terraform destroy -auto-approve
```

## Support & Feedback

This project is maintained by AWS Solution Architects. It is not part of an AWS service and support is provided best-effort basis. To post feedback, submit feature ideas, report bugs and contribute, please  see the [Contribution guide](./CONTRIBUTING.md).


## Security

See [CONTRIBUTING](./CONTRIBUTING.md#security-issue-notifications) for more information.


## License

Apache-2.0 Licensed. See [LICENSE](./LICENSE).
