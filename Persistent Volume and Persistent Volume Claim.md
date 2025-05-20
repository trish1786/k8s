## Static Provisioning

### Task 1: Get Node Label and Create Custom Index.html on Node
View worker nodes and their labels
```
kubectl get nodes --show-labels | grep role=node
```
Make a note of the kubernetes.io/hostname label of one of the nodes and ssh to one of the nodes using below command
```
ssh -t ubuntu@<node_public_IP> 
```
Switch to root and run the following commands. A directory with custom index.html is created for PersistentVolume mount 
```
sudo su
```
```
mkdir /pvdir
```
```
echo Hello World! > /pvdir/index.html
```

### Task 2: Create a Local Persistent Volume
```
vi pv-volume.yaml
```
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pvdir"
```
```
kubectl apply -f pv-volume.yaml
```
```
kubectl get pv
```
```
kubectl describe pv pv-volume
```

### Task 3: Create a PV Claim
```
vi pv-claim.yaml
```
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
```
kubectl apply -f pv-claim.yaml
```

### Task 4: Create nginx Pod with NodeSelector
```
vi pv-pod.yaml
```
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pv-pod
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
  containers:
     - name: pv-container
       image: nginx
       ports:
          - containerPort: 80
            name: "http-server"
       volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: pv-storage
  nodeSelector:
    kubernetes.io/hostname: ip-172-20-33-138.ap-south-1.compute.internal
```
Apply the Pod yaml created in the previous step
```
kubectl apply -f pv-pod.yaml
```
View Pod details and see that is created on the required node
```
kubectl get pods -o wide
```
Access shell on a container running in your Pod
```
kubectl exec -it pv-pod -- /bin/bash
```
```
exit
```
delete the resources created in this lab.
```
kubectl delete -f pv-pod.yaml
```
```
kubectl delete -f pv-claim.yaml
```
```
kubectl delete -f pv-volume.yaml
```

## Dynamic Provisioning
Creating pvc 
```
vi pv-claim.yaml
```
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: sc-claim
spec:
  storageClassName: gp2
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
```
Apply the above yaml 
```
kubectl apply -f pv-claim.yaml
```
Verify the pv and pvc
```
kubectl get pvc
```
```
kubectl get pv
```
Create deployment
```
vi ng-deploy.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ng-deploy
  name: ng-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ng-pod
  template:
    metadata:
      labels:
        app: ng-pod
    spec:
      volumes:
      - name: cloud-storage
        persistentVolumeClaim:
          claimName: sc-claim
      containers:
      - image: nginx
        name: nginx-ctr
        ports:
        - containerPort: 80
        volumeMounts:
        - name: cloud-storage
          mountPath: /usr/share/nginx/html
```
Verify deployment
```
kubectl get deploy
```
```
kubectl get pods
```


