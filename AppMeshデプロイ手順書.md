# Getting started
# Step 1:Install the integration components
  - 以下のserviceaccountを作成する
    ```bash
    kubectl create sa appmesh-controller -n appmesh-system
    ```
    > これをしないと以下のStep1にて、appmesh-controllerのreplicasがsaがないためエラーとなる
  - [Step1の手順通りに実行](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-kubernetes.html)


# Step 2:Deploy App Mesh resouces
## 1. Create a k8s namespace to deploy App Mesh resources
```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: my-apps
  labels:
    mesh: my-mesh
    appmesh.k8s.aws/sidecarInjectorWebhook: enabled
EOF
```

## 2. Create an App Mesh service mesh
```bash
cat << EOF | kubectl apply -f -
apiVersion: appmesh.k8s.aws/v1beta2
kind: Mesh
metadata:
  name: my-mesh
spec:
  namespaceSelector:
    matchLabels:
      mesh: my-mesh
EOF
```

## 3. Create an app mesh virtual node.
```bash
cat << EOF | kubectl apply -f -
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualNode
metadata:
  name: my-service-a
  namespace: my-apps
spec:
  podSelector:
    matchLabels:
      app: my-app-1
  listeners:
    - portMapping:
        port: 80
        protocol: http
  serviceDiscovery:
    dns:
      hostname: my-service-a.my-apps.svc.cluster.local
EOF
```

## 4. Create an app mesh virtual router
```bash
cat <<EOF | kubectl apply -f -
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualRouter
metadata:
  namespace: my-apps
  name: my-service-a-virtual-router
spec:
  listeners:
    - portMapping:
        port: 80
        protocol: http
  routes:
    - name: my-service-a-route
      httpRoute:
        match:
          prefix: /
        action:
          weightedTargets:
            - virtualNodeRef:
                name: my-service-a
              weight: 1
EOF
```

## 5. Create an app mesh virtual service
```bash
cat << EOF | kubectl apply -f -
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualService
metadata:
  name: my-service-a
  namespace: my-apps
spec:
  awsName: my-service-a.my-apps.svc.cluster.local
  provider:
    virtualRouter:
      virtualRouterRef:
        name: my-service-a-virtual-router
EOF
```

# Step 3:Create orupdate services

## Important
> The value for the app matchLabels selector in the spec must match the value that you specified when you created the virtual node in sub-step 3 of Step 2: Deploy App Mesh resources, or the sidecar containers won't be injected into the pod. In the previous example, the value for the label is my-app-1. If you deploy a virtual gateway, rather than a virtual node, then the Deployment manifest should include only the Envoy container. For more information about the image to use, see Envoy image. For a sample manfest, see the deployment example
on GitHub.
