# Índice
 - [Primeros comandos](#primeros-comandos)
 - [Pods](#pods)
   - [Creación por línea de comandos](#imperativo)
   - [Con YAML](#pods-con-yaml)
 - [Deployments](#deployments)
   - [Simple](#deployments)
   - [Escalado](#escalado)
 - [Services](#services)
   - [Clusterip](#escalado)
   - [Nodeport](#update)
   - [LoadBalancer](#loadbalancer)
 - [Storage](#storage)
   - [Persistent Volume](#persistent-volume)
   - [Persistent volume claim](#persistent-volume-claim)
   - [ConfigMap](#configmap)
   - [Secrets](#secrets)


## Primeros comandos
``` powershell
## set alias en powershell
Set-Alias -name k -value kubectl
```

``` powershell
k version
```

``` powershell
k cluster-info
```

``` powershell
k get all
```

### dashboard
``` powershell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```
``` powershell
k describe secret -n kube-system
```
``` powershell
k proxy
```
Ir a: 
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/


## Pods
### Crear un Pods

#### imperativo

``` powershell
kubectl run my-nginx --image=nginx:latest
```

``` powershell
kubectl get pods
```

``` powershell
kubectl get all
```

``` powershell
kubectl port-forward my-nginx 8080:80
```

``` powershell
kubectl describe pod my-nginx
```

``` powershell
kubectl delete pod my-nginx
```

``` powershell
kubectl get pods
```

#### Pods con YAML

``` yaml
apiVersion: v1
kind: Pod
metadata:
 name: my-nginx
spec:
 containers:
  - name: my-nginx
    image: nginx:alpine
```

``` powershell
kubectl apply -f .\nginx.pod.yml
```

``` powershell
kubectl delete -f .\nginx.pod.yml
```

### Pod Probe

``` yaml
apiVersion: v1
kind: Pod
metadata:
 name: my-nginx
 labels:
  app: nginx
  rel: stable
spec:
 containers:
  - name: my-nginx
    image: nginx:alpine
    ports:
     - containerPort: 80
    resources:
    livenessProbe:
     httpGet:
      path: /index.html
      port: 80
     initialDelaySeconds: 15
     timeoutSeconds: 2
     periodSeconds: 5
     failureThreshold: 1
    readinessProbe:
     httpGet:
      path: /index.html
      port: 80
     initialDelaySeconds: 15
     periodSeconds: 5
     failureThreshold: 1
```     
``` powershell
kubectl get pods --watch
```

``` powershell
kubectl exec my-nginx -it -- sh
´´´

´´´ sh
cd usr/share/nginx/html
rm index.html
```


## Deployments

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
```

``` powershell
kubectl apply -f .\deployments\nginx.deployment.yml
```

``` powershell
kubectl get all
```

``` powershell
kubectl describe deployment my-nginx
```

``` powershell
delete pod
```

``` powershell
kubectl get all
```

#### Escalado

``` powershell
kubectl scale -f .\deployments\nginx.deployment.yml --replicas=4
```

``` powershell
kubectl get pods -w
```

``` powershell
kubectl delete pod [podname]
```

``` powershell
kubectl delete deployment my-nginx
```

#### Update

``` powershell 
kubectl apply -f .\nginx.deployment.yml
```

``` powershell
kubectl get pods --watch
```

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "128Mi" #128 MB
            cpu: "200m" #200 millicpu (.2 cpu or 20% of the cpu)
        readinessProbe:
          httpGet:
            path: /index.html
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 5
          failureThreshold: 1
        livenessProbe:
          httpGet:
            path: /index.html
            port: 80
          initialDelaySeconds: 15
          timeoutSeconds: 2
          periodSeconds: 5
          failureThreshold: 1
```

``` powershell 
kubectl apply -f .\nginx.deployment.yml
```


## Services

Port forwarding

``` powershell
kubectl port-forward pod/name 8080:80
```
``` powershell
kubectl port-forward deployment/name 8080
```
``` powershell
kubectl port-forward service/name 8080
```

``` powershell
kubectl apply -f ./deployments/nginx.deployment.yml
```

``` powershell
kubectl get all
```

#### clusterIP

 ``` powershell
 k exec pod/my-nginx-5bb9b897c8-mrgds -- curl -s http://10.1.0.106 
 ```

 ``` powershell
 ## clusterip address
 k exec pod/my-nginx-5bb9b897c8-mrgds -- curl -s http://10.1.0.106 
 ```

``` yml
apiVersion: v1
kind: Service

metadata:
  type: ClusterIP
  name: nginx-clusterip ## DNS name (cluster internal)
  labels:
    app: nginx
spect:
  selector:
    app: nginx
  ports:
  - name: http
    port: 8080
    targetPort: 80
```    

 ``` powershell
 ## clusterip DNS
 k exec pod/my-nginx-5bb9b897c8-mrgds -- curl -s http://nginx-clusterip:8080
 ```

#### NodePort

``` yml
apiVersion: v1
kind: Service
metadata:
 name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: my-nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 31000
```

#### Loadbalancer

``` yml
apiVersion: v1
kind: Service
metadata:
 name: nginx-loadbalancer
spec:
 type: LoadBalancer
 selector:
    app: my-nginx
 ports:
  - name: "80"
    port: 8080
    targetPort: 80
    
  # - name: "443"
  #   port: 443
  #   targetPort: 443
  ```

``` powershell 
kubectl apply -f .\nginx.loadbalancer.yml
```

``` powershell
kubectl get all -o wide
```


## Storage

#### persistent volume

``` yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
    type: DirectoryOrCreate #por defecto es Directory, en caso del directorio no existir daría error
```

``` powershell
kubectl apply -f .\pv.yml
```

``` powershell
kubectl get pv
```

#### Persistent volume claim

``` yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

``` powershell
kubectl apply -f .\pvc.yml
```

``` powershell
kubectl get pvc -o wide
```

``` powershell
kubectl get pv -o wide
```


``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: my-nginx #importante para relacionar objetos
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-nginx #objetos a los que afecta
  template:
    metadata:
      labels:
        app: my-nginx #etiqueta custom para asociar este objeto
    spec: # definición de los Pods
      containers:
      - name: my-nginx
        image: nginx:alpine
        ports:
        resources: # acá pueden ir cosas como límites de CPU y memoria
        volumeMounts:
        - name: vol1
          mountPath: /usr/share/nginx/html        
      volumes:
      - name: vol1
        persistentVolumeClaim:
          claimName: task-pv-claim
```

## configmap


``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-configmap
spec:
  selector:
    matchLabels:
      app: nginx-configmap
  template:
    metadata:
      labels:
        app: nginx-configmap
    spec:
      containers:
      - name: nginx-configmap
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        resources:
        ports:
        - containerPort: 9000

        volumeMounts: # montamos el config map como un volumen para tenerlo accesible en un directorio del container
          - name: app-config-vol
            mountPath: /etc/config

        env: # leemos cada environment variable de a una desde un config map creado
        - name: MY_ENV1
          valueFrom:
            configMapKeyRef:
              name: app-settings
              key: MY_KEY1
              optional: true

        envFrom: # leemos todas las environment variables y las cargamos en el container
        - configMapRef:
            name: app-settings

      volumes: # se crea un volumen a partir de un config map
      - name: app-config-vol
        configMap:
          name: app-settings
```

#### secrets


``` yml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: nginx
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: db-passwords
            key: db-username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-passwords
            key: db-password
  restartPolicy: Never
```