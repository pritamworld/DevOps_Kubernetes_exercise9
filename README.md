Gotcha—sounds like you want an **nginx Deployment with 3 replicas** and a **NodePort Service** you can hit from local **Minikube**. Here’s a clean, copy-paste setup.

### 1) Manifest (recommended)

Save as `nginx-deploy-nodeport.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
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
          image: nginx:1.25-alpine
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30080   # optional; pick any free port in 30000–32767
```

Apply & verify:

```bash
kubectl apply -f nginx-deploy-nodeport.yaml
kubectl rollout status deployment/nginx
kubectl get pods -l app=nginx -o wide
kubectl get svc nginx-nodeport
```

Access it from Minikube:

```bash
# Easiest (prints a URL and opens a tunnel if needed):
minikube service nginx-nodeport --url

# Or hit the NodePort directly:
MINIKUBE_IP=$(minikube ip)
curl http://$MINIKUBE_IP:30080
```

---

### 2) One-liners (imperative alternative)

```bash
kubectl create deployment nginx --image=nginx:1.25-alpine --replicas=3
kubectl expose deployment nginx --type=NodePort --port=80 --name=nginx-nodeport
minikube service nginx-nodeport --url
```

> Note: The CLI won’t set a fixed `nodePort`; Kubernetes will auto-assign one. Use the YAML above if you want a specific port like `30080`.

---

### Cleanup

```bash
kubectl delete -f nginx-deploy-nodeport.yaml
# or, if you used the one-liners:
kubectl delete svc nginx-nodeport
kubectl delete deploy nginx
```

