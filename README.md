# Autoscale Pods based on HTTP workload using KEDA in Kubernetes
Scale Pods based on HTTP workload using KEDA &amp; keda-http-add-on

Scaling Operations based on HTTP Request

## Step 1: First we need to create a deployment/replicaset or demonset  and expose the resource using a service. Make sure to replace the namespace accordingly.
Here’s a manifest attached for reference.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: demo
spec:
  replicas: 0  # Set the number of replicas to 0
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

## Step 2: Now we need to install HELM, KEDA & KEDA http-add-on using the following commands:
> helm repo add kedacore https://kedacore.github.io/charts
> helm repo update
> helm install keda kedacore/keda --namespace keda --create-namespace
> helm install http-add-on kedacore/keda-add-ons-http --namespace keda

The HTTP Add-on is highly modular and, as expected, builds on top of KEDA core. Below are some additional components:
Operator - watches for ScaledHTTPObject CRD resources and creates necessary backing Kubernetes resources (e.g. Deployments, Services, ScaledObjects, and so forth)
Scaler - communicates scaling-related metrics to KEDA. By default, the operator will install this for you as necessary.
Interceptor - a cluster-internal proxy that proxies incoming HTTP requests, communicating HTTP queue size metrics to the scaler, and holding requests in a temporary request queue when there are not yet any available app Pods ready to serve. By default, the operator will install this for you as necessary.

Step 3: Now create a ScaledObject using the manifest provided below:

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

Step 4: Now port-forward the keda-add-ons-http-interceptor-proxy service.
Command:
kubectl port-forward svc/keda-add-ons-http-interceptor-proxy -n keda 8080:8080

Step 5: Open another terminal session and observe the pods count in the demo namespace:
watch kubectl get pods -n demo

Step 6: Now send a curl request on the exposed service to verify if the pod replica count is increasing to 1.
Command:
curl localhost:8080 -H 'Host: myhost.keda'
