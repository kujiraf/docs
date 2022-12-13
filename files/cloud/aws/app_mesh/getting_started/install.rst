Getting Started
=========================================================

Install
---------------------------------------------------------

1. Add eks-charts repository to Helm.

  .. code-block:: bash

     $ helm repo add eks https://aws.github.io/eks-charts
     $ helm repo list
       NAME    URL                             
       eks     https://aws.github.io/eks-charts

2. Install the App Mesh k8s custome resource definitions(CRD).

  .. code-block:: bash

     $ kubectl apply -k "https://github.com/aws/eks-charts/stable/appmesh-controller/crds?ref=master"

3. Create a Kubernetes namespace for the controller.

  .. code-block:: bash
     
     kubectl create ns appmesh-system

4. (Optional) Create a Fargate profile

ommit

5. Create a OIDC identity provider

  .. code-block:: bash

     $ eksctl utils associate-iam-oidc-provider \
       --region=$AWS_REGION \
       --cluster $CLUSTER_NAME \
       --approve

6. Create an IAM role

ommit. Creating with terraform.

7. Deploy the App Mesh Controller

XXXX
---------------------------------------------------------


XXXX
---------------------------------------------------------
