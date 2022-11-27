クラスタへのアクセス
=========================================================

kubectlでのアクセス
---------------------------------------------------------
* クラスタ情報を $HOME/.kube/config に追加する

  .. code-block:: bash

    export REGION=ap-northeast-1
    export CLUSTER_NAME=ma-furukawatkr-eks-cluster
    aws eks --region $REGION update-kubeconfig --name $CLUSTER_NAME

これでクラスタに対してkubectlでアクセス可能となる

AWSマネージメントコンソールからのアクセス
---------------------------------------------------------

.. note::
  「現在のユーザーまたはロールには、この EKS クラスター上の Kubernetes オブジェクトへのアクセス権がありません」と表示されてコンソール上からk8sリソースの確認ができない場合、この手順を実施する必要がある。

* ConfigMapのひな型取得

  .. code-block:: bash

    curl -o aws-auth-cm.yaml https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/aws-auth-cm.yaml

* ConfigMapを適用

  .. code-block:: bash
     
    export ACCOUNT_ID=[アカウントID]
    export NODE_ROLE=[EKS Nodeのロール]
    export IAM_USER=[AWSコンソールからアクセスするIAMユーザ名]

    cat << EOF | kubectl apply -f -
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: aws-auth
      namespace: kube-system
    data:
      mapRoles: |
        - rolearn: arn:aws:iam::${ACCOUNT_ID}:role/${NODE_ROLE}
          username: system:node:{{EC2PrivateDNSName}}
          groups:
            - system:bootstrappers
            - system:nodes
      mapUsers: |
        - userarn: arn:aws:iam::${ACCOUNT_ID}:user/${IAM_USER}
          username: ${IAM_USER}
          groups:
            - system:masters
    EOF

AWSマネージメントコンソールからk8sリソースが見れるようになっていれば完了