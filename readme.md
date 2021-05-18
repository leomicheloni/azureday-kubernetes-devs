# Índice
 - [Primeros comandos](#primeros-comandos)
 - [Pods]
   - [Creación por línea de comandos](#imperativo)
   - [Con YAML](#pods-con-yaml)


#### Primeros comandos
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
kubectl delete [podname]
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
kubectl exec pod -it -- sh
´´´

´´´ sh
cd usr/share/nginx/html
rm index.html
```


## Deployments
```
kubectl apply -f .\deployments\nginx.deployment.yml
```

```
kubectl get all
```
```
kubectl describe deployment my-nginx
```

### Escalado

```
kubectl scale -f .\deployments\nginx.deployment.yml --replicas=4
```

```
kubectl get deployment my-nginx -o yaml
```

```
kubectl get pods -w
```

```
kubectl delete pod [podname]
```

```
kubectl delete deployment my-nginx
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

[Desplegar nginx](./deployments/nginx.deployment.yml)

``` powershell
kubectl apply -f ./deployments/nginx.deployment.yml
```


``` powershell
kubectl get all
```

``` powershell
kubectl port-forward pod/my-nginx-5bb9b897c8-h4ggq 8080:80
```

``` powershell
kubectl port-forward deployment/my-nginx 8080:80
```

### clusterIP
 ``` powershell
 k exec pod/my-nginx-5bb9b897c8-mrgds -- curl -s http://10.1.0.106 
 ```

 ``` powershell
 ## clusterip address
 k exec pod/my-nginx-5bb9b897c8-mrgds -- curl -s http://10.1.0.106 
 ```

 ``` powershell
 ## clusterip DNS
 k exec pod/my-nginx-5bb9b897c8-mrgds -- curl -s http://nginx-clusterip:8080
 ```

### NodePort

``` powershell
 kubectl apply -f .\services\nginx.nodeport.service.yml
 ```


### Loadbalancer
``` powershell
 kubectl apply -f .\services\nginx.loadbalancer.service.yml
 ```


## Storage
Si un contenedor, Pod o Nodo deja de funcionar en ocasiones necesitamos tener datos que persistan y se puedan recuperar cuando el sistema se estabilice (o compartir datos al escalar).
Ese es el objetivo del storage, tener información en lugares que se pueda compatir.

### Tipos de storage
 - Volumes
 - PersistentVolumes
 - PersistentVolumeClaims
 - StorageClasses

 > Los volúmenes pueden ser utilizados para almacenar datos y estados de Pods y contenedores, y acceder después de que un Pod es re-creado


Los Pods viven y mueren, entonces su file systema es efímero
Se pueden utilizar volúmenes para almacenar datos / estado para utilizarlos en Pods
Un Pod puede tener múltiples volumnes atachados
Los contenedores utilizando **mountPath** para acceder a un volumen

> Kubernetes soporta
> - Volumes
> - PersistentVolumes
> - PersistentVolumeClaims
> - StorageClasses

### Volúmenes

Son similares a los volúmenes de Docker pero con más opciones.

 #### Tipos de volúmenes
  - emptyDir: Tiene el ciclo de vida del Pod, útil para compartir información entre los contenedores de un Pod
  - hostPath: El Pod monta el volumen sobre el file system del Nodo
  - nfs (network file system): Un volumen en algún sitio de la red que no está relacionado con el Nodo ni el cluster, tiene su propio ciclo de vida.
  - configMap/secret: Tipos especiales que permiten al Pod acceder a recursos de Kubernetes
  - persistentVolumeClaim: Permite abstraer al Pod del volument
  - Could: Sistema de permistencia fuera de la Red

  #### emptyDir
  Solo vive y es visible para un Pod

  [Ejemplo](/storage/nginx.emptydir.volume.yml)

``` powershell
k apply -f ./storage/nginx.emptydir.volume.yml
k port-forward my-nginx 8080:80
k describe pod my-nginx
```

  #### hostPath
  Se mapea el Pods
  Puede ser de diferentes tipos:
   - DirectoryOrCreate
   - Directory
   - FileOrCreate
   - File
   - Socket
   - CharDevice
   - BlockDevice

``` powershell
k apply -f ./storage/nginx.hostPath.volume.yml
k port-forward my-nginx 8080:80
k describe pod my-nginx
```

#### PersistentVolumens & PersistentVolumeClaims

Es un volumen que debe crear el administrador y depende del cluster
No depende del ciclo de vida de ningún Pod sino del cluster

 - PersistentVolume: El volumen asociado al cluster, creado por un administrador (NFS, Could, etc.)
 - PersistentVolumeClaim: El objeto que asocia un Pod o Deployment con un PersistentVolume

 #### Storage classes
 