1.  **Create eks Cluster**

eksctl create cluster --name example-cluster --region example-region --fargate
 THis will create loads of configs for our app including: Public subnet,Private subnet etc..

2.  **Download kubeconfig so we prevent visiting UI-Not a good practice**
 
 aws eks update-kubeconfig --name example-cluster --region example-region

3.  **Create a Fargate Profile so a new namespace can be created for deployment**

eksctl create fargateprofile \
    --cluster example-cluster \
    --region example-region \
    --name alb-sample-app \
    --namespace game-2048


4.  **Apply Deployment, Service and Ingress**


kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml

Detailed Files is saved under "END TO END" or open in a browser, http//raw....to view content.


run kubectl get pods -n game-2048 - to see if deployment is running
    kubectl get ingress -n game-2048 - to check ingress


5. **configure IAM OIDC provider** so other resources can communicate to aws-cloud

eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve


6. **Download iam policy**

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

You can get the above documentation from ALB Controller Documentation

7.  **Create iam policy**

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json


8. **Create iam Role**
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve


9.  **Add helm repo**
helm repo add eks https://aws.github.io/eks-charts


10.  **Update repo**
helm repo update eks


11.  **Install Helm**

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id>


12.  **Verify deployment is running**
kubectl get deployment -n kube-system aws-load-balancer-controller

Kubectl get ingress -n game-2048 - this should depict the dns address with port which can be copiied into your browser to test deployed application.



<img width="1443" height="987" alt="Screenshot (28)" src="https://github.com/user-attachments/assets/4ec85fd3-4ddb-4da7-be0a-0a4b999b3b34" />

