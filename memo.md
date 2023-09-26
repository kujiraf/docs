# k8s
## k8s network model
- Podは固有のIPを持つため、コンテナポートとホストポートのマッピングがほぼ不要
  - PodをVMや物理ホストとほぼ同様に扱えるため、後方互換性のあるマイグレが可能
- k8sのNW実装要件は以下（これでVMからPodへのマイグレが容易に行える）
  1. Podは他のノード上のすべてのPodともNAT無しで通信可能
  1. ノード上のエージェント（daemonやkubeletなど）は、そのノード上のすべてのPodと通信可能
- k8sのIPはPodスコープに存在する。Pod内のコンテナはIPアドレスとMACアドレスを含むNWネームスペースを共有するため、Pod内のコンテナ間はlocalhost上のたがいのポートに到達できる
- k8sネットワークは、以下の４つの懸念に対処する
  1. Pod内のコンテナはループバックを介して通信するためにネットワークを使用する。
  1. クラスタネットワーキングは異なるPod間の通信を提供する。
      - Service APIを使用すると、Pod内で実行されているアプリケーションをクラスタ外からアクセスできるように公開できる。
  1. aaa
  1. aaa

# クラスタ接続時に`invalid apiVersion "client.authentication.k8s.io/v1alpha1"`エラー
- EC2上で作業した際に発生
- 以下のコマンドで`~/.kube/config`に認証情報を書き込んだ
  ```bash
  export AWS_REGION=ap-northeast-1
  export CLUSTER_NAME=ma-furukawatkr-eks-cluster
  aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
  ```
- この際、AWSCLIのバージョンが古く、yamlの`apiVersion: client.authentication.k8s.io/v1beta1`が`apiVersion: client.authentication.k8s.io/v1alpha1`で書き込まれていたため、以下のエラーが発生した。
  ```bash
  invalid apiVersion "client.authentication.k8s.io/v1alpha1"
  ```
  - k8sのv1.24でv1alpha1はなくなった模様
- AWSCLIのバージョンを`2.6.3`以上にすれば解決とのことで、EC2のAWSCLI(ver1)をアンインストール後、AWSCLI 2.9.5 を入れて再度`~/.kube/config`のアップデートコマンドを実行し、解決した

# kubectlをした際に
- 事象は[この記事](https://aws.amazon.com/jp/premiumsupport/knowledge-center/eks-api-server-unauthorized-error/)の通り
- 原因
  - eksクラスタはterraformで作成しており、これは`ma_app_user`がクラスタ作成者である。そのため、クラスタ管理者も`ma_app_user`になっている
  - EC2からアクセスしようとしていたが、EC2は`aws configure`コマンドを実行しておらず、EC2にアタッチされたIAMロールでアクセスしようとしたため、拒否された
- 対策
  - EC2上で`aws configure`コマンドを実行し、`ma_app_user`のAPIキーを登録した。
    - `~/.aws/credentials`のdefaultに上記ユーザが登録されたため、何もしなければこのユーザでkubectlのアクセスとなる
- 補足
  - AWSマネージドコンソールからクラスタ情報を見る場合は、クラスタ管理者ではない`furukawatkr`で参照することになるので、`IAMUSER=furukawatkr`として以下のconfigmapを作成してあげる必要がある
    ```yaml
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

# [参考]EKSの認証認可の仕組み
https://zenn.dev/take4s5i/articles/aws-eks-authentication

# appmesh-controllerの作成
  - 以下のserviceaccountを作成する
    ```bash
    kubectl create sa appmesh-controller -n appmesh-system
    ```
    > これをしないと以下のStep1にて、appmesh-controllerのreplicasがsaがないためエラーとなる

---
# appmesh controllerとは
https://aws.github.io/aws-app-mesh-controller-for-k8s/
