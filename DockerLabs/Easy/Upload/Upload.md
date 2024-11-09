![image](https://github.com/Crisstianpd/CTFs/edit/main/DockerLabs/Easy/Upload/imgs/banner.png)
[Dockerlabs](https://dockerlabs.es/)

## Reconocimiento
Comenzamos realizando un escaneo general con **nmap** sobre la IP de la máquina víctima para ver qué puertos tiene abiertos.
Este escaneo realiza un escaneo de todos los puertos disponibles en el host "172.17.0.2", mostrando solo los
puertos abiertos, utilizando el escaneo de tipo TCP SYN ("-sT"), estableciendo una velocidad mínima de envío de paquetes de 5000 por segundo ("--min-rate 5000"), activando el modo de verbosidad extremadamente alto ("-vvv"), desactivando la resolución DNS ("-n"), no realizando el ping previo al escaneo ("-Pn"), y guardando los resultados en formato Greppable en un archivo llamado "allPorts" ("-oG allPorts").

```shell
nmap -p- --open -sT --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts
________________________________________________

PORT    STATE SERVICE REASON
80/tcp  open  http  syn-ack
```
