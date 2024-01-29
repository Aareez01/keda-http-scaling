# Autoscale Pods based on HTTP workload using KEDA in Kubernetes
Scale Pods based on HTTP workload using KEDA &amp; keda-http-add-on

# Scaling Operations based on HTTP Request

## Step 1: Deploy Nginx with Zero Replicas

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
