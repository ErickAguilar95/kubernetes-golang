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
// main.go
package main

import (
    "encoding/json"
    "fmt"
    "net"
    "net/http"
)

type Response struct {
    Message string `json:"message"`
}

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        ifaces, err := net.Interfaces()
        if err != nil {
            panic(err)
        }

        var response Response
        var ip string

        for _, i := range ifaces {
            addrs, err := i.Addrs()
            if err != nil {
                panic(err)
            }

            for _, address := range addrs {
                if ipnet, ok := address.(*net.IPNet); ok && !ipnet.IP.IsLoopback() {
                    if ipnet.IP.To4() != nil {
                        ip = fmt.Sprintf("Hello response by %s", ipnet.IP.String())
                    }
                }
            }
        }

        response.Message = ip
        fmt.Println(response)
        json.NewEncoder(w).Encode(response)
    })

    http.ListenAndServe(":8000", nil)
}
```

Vamos a crear el contenedor para que construya y corra nuestra aplicacion golang
```
# DockerFile
```



docker build -t project-app:latest .

docker tag project-app:latest aguila95/project-app:latest

docker push aguila95/project-app:latest

```
minikube start --mount --mount string="/home/erickaja/infotec/kubernetes:/home/app"
```
