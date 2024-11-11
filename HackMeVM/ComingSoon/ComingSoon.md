
![image]()

[Maquina](https://hackmyvm.eu/machines/machine.php?vm=Comingsoon)   \   [HackMeVM](https://hackmyvm.eu/)


## Descubrimiento
Primeramente, comenzamos descubriendo cual es la **IP** de la maquina vulnerable, para ello, podemos hacer uso de herramientas como `arp` o `nmap`.

Con `arp` podemos hacer uso del parametro `-a` para listar las *IPs* y *direcciones MAC* de los dispositivos recientemente conectados.
```shell
arp -a
____________
...
? (10.10.10.6) at 08:00:27:67:5e:78 [ether] on eth0
...
```
Con `nmap` podemos hacer uso del parametro `-sn` con la direccion que representa a la red que en mi caso es la `10.10.10.0` junto con un `/24` que es una notacion en *CIDR* que representa el rango de *IPs* que abarca desde **10.10.10.1** hasta la **10.10.10.254**. Esto hara que `nmap` envie paquetes *ICMP* a cada direccion *IP* del rango antes mecionado, en caso de que una IP envie de regreso un paquete *ICMP* o una respuesta de *TCP/ACK*, nmap considera que el dispositivo esta activo.
```shell
nmap -sn 10.10.10.0/24
__________________________________________
...
Nmap scan report for 10.10.10.6
Host is up (0.0013s latency).
MAC Address: 08:00:27:67:5E:78 (Oracle VirtualBox virtual NIC)
...
```
Al nosotros conocer cual es la direccion **IP** de nuestra maquina vulnerable que, en mi caso, es la **IP** `10.10.10.6` podemos dar comienzo a la fase de escaneo de puertos.



## Reconocimiento
Comenzamos con un escaneo de `nmap` a todos los puertos de la maquina vulnerable en busca de puertos abiertos.

```shell
nmap -p- --open -sS --min-rate 5000 -vvv 10.10.10.6 -oG allPorts
________________________________________________________________
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64

```

Nmap nos reporta que los puertos ***22*** y ***80*** estan abiertos. El puerto **22** corresponde al servicio de conexiones via `ssh` o *Secure Shell* que es un protocolo de red que permite acceder y administrar de forma remota otros dispositivos de manera segura. Por otro lado, el puerto **80**, es un puerto por defecto que corresponde a un servicio `http`(Hyper Text Transfer Protocol) o a un servicio web en otras palabras.

Podriamos tratar de conseguir solo una de las dos credenciales necesarias para ingresar a la maquina por `SSH` con fuerza bruta pero, sigamos recopilando informacion.

Ahora hacemos un segundo escaneo con `nmap` sobre este puerto abierto con un conjunto de scripts basicos de reconocimientos que trae por defecto nmap con el parametro `-sC` que nos brindara informacion y detalles sobre los puertos, he incluimos el parametro `-sV` para que nos detecte la version del servicio que corre en dicho puerto. Estos dos se pueden escribir como `-sCV`.

```shell
nmap -p22,80 -sCV 10.10.10.6 -oN targeted
_________________________________________________________________
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   3072 bc:fb:ec:b8:93:d4:e2:78:76:eb:1b:dc:4b:a7:7f:9b (RSA)
|   256 31:41:a0:d7:e9:3c:79:11:c2:f0:81:a0:fe:2d:f9:b0 (ECDSA)
|_  256 c9:34:17:00:31:75:4d:c0:3a:a5:b1:16:36:0d:bb:18 (ED25519)
80/tcp open  http    Apache httpd 2.4.51 ((Debian))
|_http-title: Bolt - Coming Soon Template
|_http-serve r-header: Apache/2.4.51 (Debian)

```
Nmap no nos reporta mucha informacion relevante por lo que abriremos nuestro navegador y colocaremos la *IP* de la maquina vulnerable para poder visualizar la pagina web que corre dentro de la maquina y buscar posibles vectores de ataque.
![image]()

Dentro de la web y en su codigo no encontramos nada interesante.

Usamos `whatweb` para determinar las tecnologias y sus versiones que son usadas en la pagina web y poder determinar si alguna de estas desactualizada y si contiene alguna vulnerabilidad conocida.
```shell
whatweb 10.10.10.6
____________________________________
http://10.10.10.6 [200 OK] Apache[2.4.51], Bootstrap, Cookies[RW5hYmxlVXBsb2FkZXIK], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.51 (Debian)], IP[10.10.10.6], JQuery, Script[text/javascript], Title[Bolt - Coming Soon Template]

```
Con `whatweb` tampoco nos encontramos con algo interesante asi que haremos uso de `gobuster` para buscar directorios y archivos que se encuentran alojadas en el mismo servidor *http*.

A `gobuster`, con el parametro `-x` le indicamos que extensiones de archivos deceamos comprobar su existencia usando del mismo diccionario que se usara para descubrir los posibles directorios dentro del servidor indicado por el parametro `-w`, y con `--url` le indicamos la direccion de la pagina web.
```shell
 gobuster dir --url http://10.10.10.6 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -x xml,txt,php,html
____________________________________________________________________________________________________
...
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 3988]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/assets               (Status: 301) [Size: 309] [--> http://10.10.10.6/assets/]
/license.txt          (Status: 200) [Size: 528]
/notes.txt            (Status: 200) [Size: 279]
...
```











