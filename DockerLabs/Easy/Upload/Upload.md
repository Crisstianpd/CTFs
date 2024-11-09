![image](https://github.com/Crisstianpd/CTFs/blob/13e06aa9608e2ab7b5a6bc12b876802724de4e3a/DockerLabs/Easy/Upload/imgs/banner.png)

[Dockerlabs](https://dockerlabs.es/)

## Reconocimiento
Comenzamos con un escaneo de nmap a todos los puertos de la maquina vulnerable en busca de puertos abiertos.

```shell
nmap -p- --open -sS --min-rate 5000 -vvv 172.17.0.2 -oG allPorts
________________________________________________________________

PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
```

El puerto 80 corresponde a un servicio de http(Hyper Text Transfer Protocol) o servicio web en pocas palabras.
Ahora hacemos un escaneo sobre el unico puerto que se encuentra abierto(puerto 80) con un conjunto de scripts basicos de reconocimiento que trae por defecto nmap.
```shell
nmap -p80 -sCV 172.17.0.2 -oN targeted
______________________________________
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)
```
Guiandonos por el escaneo podemos obersvar que probablemente nos encontraremos con la oportunidad de subir archivos al servidor desde una web especifica para ello.
Es hora de visualizar la pagina web.

