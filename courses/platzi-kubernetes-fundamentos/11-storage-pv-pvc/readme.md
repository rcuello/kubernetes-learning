# 11-storage-pv-pvc

## First, create the directory for hostPath, remember to run this on the minikube node using sudo
```bash
minikube ssh
echo '<h1>Hello from Volume!</h1>' > /mnt/data/index.html
```

## Apply the PV and PVC
```bash
kubectl apply -f pv-pvc.yaml
```

## Create the Pod
```bash
kubectl apply -f pod.yaml
```

## Verify the setup
```bash
# Check PV and PVC status
kubectl get pv,pvc

# Check pod status
kubectl get pod my-pod

# Verify the mounted content
kubectl exec my-pod -- ls -la /usr/share/nginx/html

# Test the nginx server

kubectl port-forward my-pod 8080:80
```

Then you can visit http://localhost:8080 in your browser to see the content.

## Cleanup
```bash
kubectl delete pod my-pod
kubectl delete -f pv-pvc.yaml
```
