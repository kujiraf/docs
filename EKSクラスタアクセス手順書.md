# 踏み台からのkubectlアクセス
- EKS nodeのセキュリティグループのインバウンドに、踏み台のセキュリティグループからのアクセスを許可する

# kubeconfigのアップデート
```bash
export AWS_REGION=ap-northeast-1
export CLUSTER_NAME=ma-furukawatkr-eks-cluster
aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
```

# コンソールからのアクセス
- 以下のコマンドで、コンソールからのアクセス（正確には IAM User `furukawatkr`でクラスタへの認証）が可能。
- IAM User `furukawatkr`は、クラスタとして認証されるIAM Role `MaFurukawatkrEksNodeRole`にassume roleしてアクセスできると思われる（要明確化）

```bash
export ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)
export NODE_ROLE=MaFurukawatkrEksNodeRole
export IAM_USER=furukawatkr

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
```

# EBS CSI driverの有効化
https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html
- `Container Storage Interface (CSI)`は、PVとしてのEBSのライフサイクルを管理するもの
- `appmesh-prometheus`では、TSDB用のPVが必要。pvプロビジョニングのためにCSIドライバの有効化を行う。
> 注）CSI driver用のIRSAが必要

### 1. Create EBS CSI plugin IAM role with eksctl
```bash
export CLUSTER_NAME=ma-furukawatkr-eks-cluster
export EBS_CSI_DriverRoleName=MaFurukawatkrEbsCsiDriverRole

eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster $CLUSTER_NAME  \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name $EBS_CSI_DriverRoleName
```

### 2. Adding the Amazon EBS CSI add-on
```bash
eksctl create addon --name aws-ebs-csi-driver --cluster $CLUSTER_NAME --service-account-role-arn arn:aws:iam::${ACCOUNT_ID}:role/$EBS_CSI_DriverRoleName --force
```

# appmesh-prometheusのインストール
- あらかじめ上記のCSIドライバのadd-onを追加しておく必要あり
- 以下の手順でインストール
  - https://github.com/aws/eks-charts/tree/master/stable/appmesh-prometheus


# Installing the Kubernetes Metrics Server
## `top`コマンド有効化のためにMetrics Serverをインストールする
https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
