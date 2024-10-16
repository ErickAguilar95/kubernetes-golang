# Ejemplo Kubernetes con Golang

## Requisitos del proyecto
---

- [x] Docker
- [x] Minikube
- [x] Kubernetes
- [x] kubectl

## ¿Qué vamos a hacer?
---
Crearemos un clúster de Kubernetes utilizando Minikube como clúster local. También utilizaremos un pequeño proyecto de Golang y Docker como endpoint y veremos cómo balancea las cargas Kubernetes.

## ¿Qué es Kubernetes?
---
Kubernetes es una plataforma de código abierto diseñada para automatizar la implementación, el escalado y la gestión de aplicaciones en contenedores. Permite orquestar múltiples contenedores distribuidos en un clúster de servidores, proporcionando herramientas para equilibrar la carga, gestionar el almacenamiento, supervisar el estado de los contenedores y garantizar que las aplicaciones se mantengan en funcionamiento de manera eficiente.

## Conceptos básicos de Kubernetes
---
### Deployment
Un Deployment es un objeto que se utiliza para gestionar la implementación y el ciclo de vida de aplicaciones en contenedores. Proporciona una forma declarativa de definir cómo deben ejecutarse y gestionarse las aplicaciones, especificando detalles como la imagen del contenedor, el número de réplicas (pods) que deben estar corriendo y cómo realizar actualizaciones o cambios sin interrumpir el servicio.

Los Deployments permiten:

- Escalar horizontalmente las aplicaciones ajustando el número de réplicas.
- Realizar actualizaciones y reversión de versiones de forma controlada.
- Monitorear y reiniciar automáticamente los contenedores que fallan.
- Garantizar que siempre haya un estado deseado en los pods, aplicando cambios cuando sea necesario para mantener ese estado.

### Pod
Un Pod es la unidad más pequeña y básica de despliegue en la plataforma, que encapsula uno o más contenedores que comparten recursos como red y almacenamiento. Cada Pod representa una instancia de aplicación o componente, y puede contener un único contenedor (el caso más común) o varios contenedores que necesitan colaborar estrechamente.

Características clave de un Pod:

- **Compartición de recursos**: Los contenedores dentro de un pod comparten la misma dirección IP, puertos y espacio de almacenamiento (si está configurado), lo que les permite comunicarse de manera eficiente.
- **Escalabilidad**: Kubernetes escala las aplicaciones no mediante contenedores individuales, sino replicando pods a través de controladores como los Deployments.
- **Ciclo de vida**: Los pods son efímeros, lo que significa que no están destinados a durar para siempre. Kubernetes puede crear, eliminar o recrear pods según sea necesario para mantener el estado deseado de una aplicación.

Aunque los contenedores dentro de un pod comparten red y almacenamiento, son independientes en cuanto a su ejecución y pueden comunicarse entre ellos a través de los recursos compartidos.

### NodePort
NodePort es un tipo de servicio que expone un puerto específico de cada nodo del clúster, permitiendo que las aplicaciones sean accesibles desde fuera del clúster. Básicamente, Kubernetes asigna un puerto en el rango 30000–32767 a cada nodo, y cualquier solicitud que llegue a ese puerto se redirige al servicio dentro del clúster.

Cuando un servicio se define como NodePort, actúa de manera similar a un load balancer rudimentario porque Kubernetes distribuye el tráfico entrante entre los pods detrás de ese servicio. Esto se logra mediante la redirección a través del puerto asignado en cualquier nodo del clúster. Sin embargo, a diferencia de un servicio de tipo LoadBalancer, NodePort no proporciona un balanceo de carga avanzado ni configuración automática con proveedores de nube.

*Sin embargo, si estás en un entorno en la nube (AWS, GCP, Azure, etc.), puedes usar un servicio de tipo LoadBalancer que balanceará el tráfico entre los pods. En Minikube, no tendrás un balanceador de carga externo, pero puedes simular el comportamiento de un LoadBalancer usando el servicio de tipo NodePort.*

## Minikube
---
Minikube es una herramienta que permite ejecutar un clúster de Kubernetes local en una sola máquina, como un entorno de desarrollo. Es ideal para aprender y probar Kubernetes sin necesidad de desplegar un clúster completo en un proveedor de nube o una infraestructura compleja.

### Instalación
Para la instalación dependerá de cada Sistema Operativo. Para este ejemplo usaremos WSL Ubuntu.
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```
Para más información sobre Minikube, puedes visitar la [documentación oficial](https://minikube.sigs.k8s.io/docs/start).

### Comandos básicos
Este comando iniciará un clúster de manera local en una máquina virtual.
```bash
$ minikube start
```

Este comando detendrá la ejecución de la máquina virtual dejando de consumir recursos.
```bash
$ minikube stop
```

Si vamos a trabajar con volúmenes y queremos pasar archivos a nuestros contenedores:
* Minikube al ser una máquina virtual se ejecuta en paralelo a nuestro equipo local, así que nuestros archivos son independientes a los de la máquina virtual.
* Hay que montar un volumen (para el caso de Docker) con el cual compartir archivos entre nuestro equipo local y la máquina virtual Minikube.
* Con este comando indicaremos el volumen a crear para poder compartir directorios.
```bash
$ minikube start --mount --mount-string="/path/local:/path/en/minikube"
```

Un dashboard accesible desde un URL en la cual podremos ver nuestro clúster de manera gráfica.
```bash
$ minikube dashboard
```

Conocer la IP de Minikube, esto nos servirá para poder consultar nuestros servicios.
```bash
$ minikube ip
```

## Ejecución de nuestro primer clúster
---
<small>**Los outputs podrían variar según las características de cada equipo**</small>

Vamos a crear nuestro primer clúster con Minikube.
```bash
$ minikube start
``` 
```text
😄  minikube v1.34.0 on Ubuntu 24.04 (amd64)
✨  Automatically selected the docker driver
📌  Using Docker driver with root privileges
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.45 ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.31.0 on Docker 27.2.0 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

❗  /usr/local/bin/kubectl is version 1.16.0, which may have incompatibilities with Kubernetes 1.31.0.
    ▪ Want kubectl v1.31.0? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Como podemos ver, no tenemos hasta ahora ningún otro servicio más que el que crea Minikube con Kubernetes.
```bash
$ kubectl get all -o wide 
```
```text
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3m6s   <none>
```

Ahora vamos a aplicar el deployment.
```bash
$ kubectl apply -f deployment.yaml
```
```text
deployment.apps/golang-deployment created
```

Si consultamos de nuevo el listado de servicios, podremos ver cómo ya tenemos tanto los pods con el número de réplicas indicado y los servicios de deployment.
```bash
$ kubectl get all -o wide 
```
```text
NAME                                     READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod/golang-deployment-74cfbc77b9-svzjd   1/1     Running   0          12s   10.244.0.4   minikube   <none>           <none>
pod/golang-deployment-74cfbc77b9-wd72p   1/1     Running   0          12s   10.244.0.3   minikube   <none>           <none>

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10m   <none>

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES                              SELECTOR
deployment.apps/golang-deployment   2/2     2            2           12s   golang-kube   aguila95/golang-simple-api:latest   app=golang-app

NAME                                           DESIRED   CURRENT   READY   AGE   CONTAINERS    IMAGES                              SELECTOR
replicaset.apps/golang-deployment-74cfbc77b9   2         2         2       12s   golang-kube   aguila95/golang-simple-api:latest   app=golang-app,pod-template-hash=74cfbc77b9
```

Es hora de aplicar el Servicio de NodePort para poder consultarlo desde nuestra máquina local al puerto indicado.
```bash
$ kubectl apply -f serviceNodePort.yaml
```
```text
service/golang-loadbalancer created
```

Ahora vamos a consultar la IP de Minikube para poder consultar el puerto asignado de nuestra aplicación.
```bash
$ minikube ip
```
```text
192.168.49.2
```

Ahora podemos hacer un curl a nuestro clúster para poder ver cómo reparte las solicitudes entre los diferentes pods.
Para validar que se balancea la carga, la aplicación imprime la IP del pod, así que podremos ver cómo la IP cambia con cada petición.

```bash
$ curl 192.168.49.2:30001
{"message":"Hello response by 10.244.0.3"}
```
```bash
$ curl 192.168.49.2:30001
{"message":"Hello response by 10.244.0.4"}
```
```bash
$ curl 192.168.49.2:30001
{"message":"Hello response by 10.244.0.3"}
```
```bash
$ curl 192.168.49.2:30001
{"message":"Hello response by 10.244.0.4"}
```
```bash
$ curl 192.168.49.2:30001
{"message":"Hello response by 10.244.0.4"}
```
## Conclusión
---
En este ejemplo, hemos demostrado cómo crear un clúster de Kubernetes local utilizando Minikube y desplegar una pequeña aplicación Go para ver cómo Kubernetes maneja el escalado y el balanceo de carga

## Referencias
---

- [Repositorio del proyecto Golang Simple API](https://github.com/ErickAguilar95/golang-simple-api.git)