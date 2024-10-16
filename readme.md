# Ejemplo Kubernetes con Golang

## Requisitos del proyecto
---

- [x] Docker
- [x] Kubernetes
- [x] Minukube

## Docker

## Construccion del proyecto
---

Vamos a crear una aplicacion en golang la cual sera una api muy sensilla que nos responda la ip de la maquina.

```
```

Vamos a crear el contenedor para que construya y corra nuestra aplicacion golang
```
# DockerFile
```



docker build -t project-app:latest .

docker tag project-app:latest aguila95/project-app:latest

docker push aguila95/project-app:latest

```
minikube start --mount --mount string="/home/erickaja/projects/kubernetes-golang:/home/app"
```


// Todo
