#### Step by step deployment of an application on AWS EKS
1. Create an AWS IAM service role for both EC2 and EKS Cluster
2. Provision an EC2 instance and attach the instance profile provisioned above
3. Install kubectl and ekctl
*****************************
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
sudo chmod +x ./kubectl
sudo mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
****
4. create EC2 key pair to used by the node
#aws ec2 create-key-pair --region us-west-2 --key-name myKeyPair
5. Provision the cluster by executing the below commands

eksctl create cluster \
--name my-cluster \
--region <name-of-region> \
--with-oidc \
--ssh-access \
--ssh-public-key <your-key> \
--managed

6. Once you setup kubectl and AWS-CLI in your machine run below command to configure kubectl for AWS EKS & clone this repository

aws eks --region <region> update-kubeconfig --name <clusterName>

7. Now we need to create worker nodes to join the Kubernetes cluster. To create them, navigate again to AWS CloudFormation and click on “Create stack”.
Give the below URL as the Amazon S3 URL and click Next.

https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/amazon-eks-nodegroup.yaml

8. Edit the contents of aws-auth-cm.yaml to have the node arn profile by running
vi aws-auth-cm.yaml
9. Save the file and move it to the below destination and execute to join the nodes inthe cluster

mv ~/.kube/
kubectl apply -f ~/.kube/aws-auth-cm.yaml

10. Launch your app in Kubernetes executing the following

kubectl apply -f nginx.yaml
kubectl apply -f nginx-service.yaml

11. Now get the details of the running helloworld service in your cluster to obtain the endpoint

kubectl get svc service-helloworld -o yaml