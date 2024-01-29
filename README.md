# Autoscale Pods based on HTTP workload using KEDA in Kubernetes
Scale Pods based on HTTP workload using KEDA &amp; keda-http-add-on

# Scaling Operations based on HTTP Request

## Step 1: Create a Deployment/Statefulset or DaemonSet with Zero Replicas but for now we're using nginx for the example.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: demo
spec:
  replicas: 0
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: demo
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

# Installing HELM, KEDA, and KEDA HTTP Add-on

## Step 2: Install HELM

Make sure you have HELM installed. If not, you can follow the official documentation for installation: [HELM Installation Guide](https://helm.sh/docs/intro/install/)

## Step 3: Install KEDA & KEDA HTTP Add-on

```bash
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace
helm install http-add-on kedacore/keda-add-ons-http --namespace keda
```

# HTTP Add-on Components

The HTTP Add-on for KEDA is highly modular and builds on top of KEDA core. Below are the key components:

## Operator

The Operator watches for ScaledHTTPObject CRD (Custom Resource Definition) resources and creates necessary backing Kubernetes resources. These resources may include Deployments, Services, ScaledObjects, and other components required for scaling.

## Scaler

The Scaler is responsible for communicating scaling-related metrics to KEDA. By default, the operator will install the Scaler as necessary, ensuring that the scaling process functions smoothly.

## Interceptor

The Interceptor is a cluster-internal proxy designed to handle incoming HTTP requests. It communicates HTTP queue size metrics to the Scaler and holds requests in a temporary queue when there are no available app Pods ready to serve. By default, the operator will install the Interceptor as necessary.

Adjust the paths and namespaces according to your specific setup.

**Note:** Make sure to configure paths and namespaces based on your specific environment and requirements.


# Step 3: Create a ScaledObject

Now, create a ScaledObject using the manifest provided below. This ScaledObject configuration is designed for scaling based on HTTP requests.

```yaml
kind: HTTPScaledObject
apiVersion: http.keda.sh/v1alpha1
metadata:
    name: nginx-scaledobject
    namespace: demo
spec:
    hosts:
    - myhost.keda
    pathPrefixes:
    - /
    scaleTargetRef:
        deployment: nginx-deployment
        service: nginx-service
        port: 80
    replicas:
        min: 0
        max: 10
    scaledownPeriod: 20
```

# Step 4: Port-forward the Interceptor Proxy Service

Port-forward the `keda-add-ons-http-interceptor-proxy` service to observe the HTTP requests.

```bash
kubectl port-forward svc/keda-add-ons-http-interceptor-proxy -n keda 8080:8080
```

# Step 5: Monitor Pod Count

Open another terminal session and observe the pod count in the `demo` namespace using the `watch` command.

```bash
watch kubectl get pods -n demo
```

# Step 6: Verify Scaling with a Curl Request

Now, send a curl request on the exposed service to verify if the pod replica count is increasing to 1.

```bash
curl localhost:8080 -H 'Host: myhost.keda'
```

