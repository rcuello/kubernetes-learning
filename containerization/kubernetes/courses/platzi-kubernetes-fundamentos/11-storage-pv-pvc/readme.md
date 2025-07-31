# 11-storage-pv-pvc


## First, create the directory for hostPath, remember to run this on the minikube node using sudo

```bash
minikube ssh
sudo su
# Solo se puede manipular la carpeta mnt siendo administrador por eso se usa "sudo su"
mkdir /mnt/nginx-pv-data
echo '<h1>Hello from Volume!</h1>' > /mnt/nginx-pv-data/index.html
ls /mnt/nginx-pv-data/

cat /mnt/nginx-pv-data/index.html

#Salir de minikube ssh
exit
```

## Apply the PV and PVC
```bash
kubectl apply -f pv-pvc.yaml
```

Salida: 

```
    persistentvolume/my-pv created
    persistentvolumeclaim/my-pvc created
```

## Create the Pod
```bash
kubectl apply -f pod.yaml
```

Salida: 

```
    pod/my-pod created
```

## Verify the setup
```bash
# Check PV and PVC status
kubectl get pv,pvc
```

Salida: 
```
NAME                     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/my-pv   1Gi        RWO            Retain           Bound    default/my-pvc   manual         <unset>                          117s

NAME                           STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/my-pvc   Bound    my-pv    1Gi        RWO            manual         <unset>                 117s
```


```bash
# Check pod status
kubectl get pod my-pod
```

Salida:
```
NAME     READY   STATUS    RESTARTS   AGE
my-pod   1/1     Running   0          2m28s
```

```bash
# Verificar unidad montada
kubectl describe pod my-pod
```

Al ver la informacion del POD ser pueden observar las referencias del almacenamiento:
```
Name:             my-pod
Namespace:        default
...
IPs:
  IP:  10.244.0.75
Containers:
  my-container:
    ...
    Image:          nginx
    ...
    Mounts:
      /usr/share/nginx/html from my-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qvrmr (ro)
...
Volumes:
  my-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  my-pvc
    ReadOnly:   false

```

## Ingresamos directamente al pod para verificar el archivo html
```bash
# Verify the mounted content
kubectl exec my-pod -- ls -la /usr/share/nginx/html

# Si usas gitbash(windows) usa este comando (Esto fuerza a `sh` dentro del pod a interpretar el comando, no Git Bash.):
kubectl exec my-pod -- sh -c "ls -la /usr/share/nginx/html"
kubectl exec my-pod -- sh -c 'ls -la /usr/share/nginx/html'
```

Salida:
```
total 24
drwxr-xr-x 2 root root  4096 Jul 31 19:37 .
drwxr-xr-x 3 root root  4096 Jul 22 01:13 ..
-rw-r--r-- 1 root root 12288 Jul 31 19:38 .index.html.swp
-rw-r--r-- 1 root root    28 Jul 31 19:37 index.html
```

```bash
# Test the nginx server

kubectl port-forward my-pod 8080:80
```

Salida en consola:

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080
```

Then you can visit http://localhost:8080 in your browser to see the content:

```
Hello from Volume!
```



## Cleanup
```bash
kubectl delete pod my-pod
kubectl delete -f pv-pvc.yaml
```
