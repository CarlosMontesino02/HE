# Informe sobre Escaneo de Puertos de un Dispositivo

## 1. Introducción
   ### 1.1 Objetivo del informe
   ### 1.2 Contexto del escaneo
## 2. Metodología de Escaneo de Puertos
   ### 2.1 Herramienta utilizada
   ### 2.2 Alcance del escaneo
## 3. Resultados del Escaneo
   ### 3.1 Detección del dispositivo y su sistema operativo
   ### 3.2 Puertos abiertos
   ### 3.3 Scripts y análisis
## 4. Resumen de hallazgos clave
## 5. Anexos
 ### a) Identifica la IP del equipo objetivo.
 ### b) Identifica Sistema operativo.
 ### c) Identifica puertos/servicios abiertos (TCP / UDP).
 ### d) Identifica versiones de los servicios detectados.
 ### e) Comprueba si existen usuarios con contraseñas vacías (NSE).
 ### f) Comprueba las vulnerabilidades existentes en el equipo (NSE).
 ### g) Comprueba si dispone de servicios web habilitados (NSE).
 ### h) Ejecuta los scripts por defecto de nmap para ampliar la información (NSE).

---

# Introducción

Como participante en la realización de una auditoría de caja negra, se me ha encargado la tarea de recopilar toda la información posible de un servidor corporativo mediante técnicas activas (NMAP), con el objetivo de determinar los posibles vectores de ataque.
## 1.1 Objetivo del informe

Por lo tanto entre los objetivos del informe tenemos:

- Determinar posibles vectores de ataque
- Brindar una visión clara del estado del sistema analizado y servicios expuestos
- Demostrar el uso de la herramienta *nmap*
# 1.2 Contexto del escaneo

En este caso se va a realizar un escaneo a una máquina virtual, siendo esta Metasploitable 3,   es una máquina virtual construida desde cero con una gran cantidad de vulnerabilidades de seguridad, al ser una auditoria de caja negra me limito a esta información. El software de virtualización usado es Oracle VirtualBox, la red está configurada en modo "solo anfitrión". El sistema operativo anfitrión es un Kali Linux 6.5.0-kali3-amd64.

# 2. Metodología de Escaneo de Puertos

## 2.1 Herramienta utilizada

La herramienta utilizada en esta práctica es **Nmap version 7.94SVN ( https://nmap.org )**, es una herramienta de código abierto utilizada para el descubrimiento y escaneo de redes, así como para auditar la seguridad de sistemas informáticos. Proporciona información detallada sobre los hosts de una red, incluyendo los servicios y puertos abiertos, lo que facilita el análisis de la seguridad y la configuración de los sistemas.
## 2.2 Alcance del escaneo

El alcance de este escaneo comprende solo el host a escanear mencionado anteriormente, pero dentro del rango de puertos de este host, no existen límites más que el propio límite de puertos existente (desde el 0 al 65535).

## 3. Resultados del Escaneo
## 3.1 Detección del dispositivo y su sistema operativo

El primer paso es identificar la dirección IPv4 del dispositivo en nuestra red, para ello usaremos el siguiente comando:

``` sh
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap -sn 192.168.56.0/24         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 19:48 CET  
Nmap scan report for 192.168.56.1  
Host is up (0.0023s latency).  
Nmap scan report for 192.168.56.8  ----> Aquí nos indica el reporte de esta IP
Host is up (0.00092s latency).  ----> Como vemos el host está disponible.
Nmap done: 256 IP addresses (2 hosts up) scanned in 15.94 seconds
```

En el comando anterior, la opción -sn hace un escaneo usando el protocolo ICMP, resulta bastante más rápido que la opción -sL, que lista los objetivos en la red, pero tarda mucho más. Además en este caso no usamos la dirección de un dispositivo, si no la dirección de la red entre la máquina virtual y nuesto host anfitrión, **192.168.56.0/24**. Como vemos la IP encontrada es la 192.168.56.8, ya que la ip 192.168.56.1 es la de nuestro dispositivo.

Ahora vamos a proceder a identificar el sistema operativo del dispositivo, para ello usamos la opción -O.

``` sh
┌──(carlosmofe㉿reaper)-[~]  
└─$ sudo nmap -O 192.168.56.8     
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 19:51 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.00039s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
80/tcp   open   http  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
3000/tcp closed ppp  
3306/tcp open   mysql  
8080/tcp open   http-proxy  
8181/tcp closed intermapper  
MAC Address: 08:00:27:09:B5:04 (Oracle VirtualBox virtual NIC)  
Aggressive OS guesses: Linux 3.2 - 4.9 (98%), Linux 3.10 - 4.11 (94%), Linux 3.13 (94%), Linux 3.13 - 3.16 (94%), OpenWrt Chaos Calmer 15.05 (Linux 3.18) or Designated Driver (Linux 4.1 or 4.4) (94%), Linux 4.10 (94%), Android 5.0 - 6.0.1 (Linux 3.4) (94%), Linux 3.10 (94%), Linux 3.2 - 3.10 (94%), Linux 3.2 - 3.16 (94%)  
No exact OS matches for host (test conditions non-ideal).  
Network Distance: 1 hop  
  
OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 21.49 seconds
```

Este comando nos ha dado un listado de puertos con su estado y servicio, pero en este apartado nos fijamos en la parte inferior, donde vemos "Aggressive OS guesses", en esta sección se nos muestran diferentes versiones del sistema operativo y su porcentaje de similitud. Vemos que Nmap nos da un 94% de similitud con Linux 3.13, un 98% en el rango de Linux 3.2 a 4.9 etc. Como vemos en este caso no es muy precisa ya que solo hemos obtenido un abanico de posibilidades pero necesitaríamos más herramientas para determinar correctamente el sistema operativo.
## 3.2 Puertos abiertos

Para identificar los puertos abiertos, hemos usado nmap indicándole que busque entre todos los puertos (con la opción -p-) y nos informe sobre lo que ha encontrado.

```sh
┌──(carlosmofe㉿reaper)-[~]  
└─$ sudo nmap -p- 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 19:54 CET  
Stats: 0:01:25 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan  
SYN Stealth Scan Timing: About 61.97% done; ETC: 19:57 (0:00:45 remaining)  
Nmap scan report for 192.168.56.8  
Host is up (0.00027s latency).  
Not shown: 65524 filtered tcp ports (no-response)  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
80/tcp   open   http  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
3000/tcp closed ppp  
3306/tcp open   mysql  
3500/tcp open   rtmp-port  
6697/tcp open   ircs-u  
8080/tcp open   http-proxy  
8181/tcp closed intermapper  
MAC Address: 08:00:27:09:B5:04 (Oracle VirtualBox virtual NIC)  
  
Nmap done: 1 IP address (1 host up) scanned in 117.10 seconds
```

Como vemos los puertos abiertos son:

| PORT     | STATE   | SERVICE       |
|----------|---------|---------------|
| 21/tcp   | open    | ftp           |
| 22/tcp   | open    | ssh           |
| 80/tcp   | open    | http          |
| 445/tcp  | open    | microsoft-ds  |
| 631/tcp  | open    | ipp           |
| 3000/tcp | closed  | ppp           |
| 3306/tcp | open    | mysql         |
| 3500/tcp | open    | rtmp-port     |
| 6697/tcp | open    | ircs-u        |
| 8080/tcp | open    | http-proxy    |
| 8181/tcp | closed  | intermapper   |

Si nos preguntamos como es posible que nmap devuelva los servicios de puertos cerrados, esto podría deberse a que un firewall o una configuración de servicio bloquea el tráfico pero aún así el servicio se encuentra configurado y responde algunas solicitudes de nmap.

Si queremos ver los servicios y sus versiones más a fondo usaremos la opción -sV, de forma que la respuesta es :
``` sh
┌──(carlosmofe㉿reaper)-[~]  
└─$ sudo nmap -sV 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 19:53 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.00028s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE     VERSION  
21/tcp   open   ftp         ProFTPD 1.3.5  
22/tcp   open   ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)  
80/tcp   open   http        Apache httpd 2.4.7  
445/tcp  open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
631/tcp  open   ipp         CUPS 1.7  
3000/tcp closed ppp  
3306/tcp open   mysql       MySQL (unauthorized)  
8080/tcp open   http        Jetty 8.1.7.v20120910  
8181/tcp closed intermapper  
MAC Address: 08:00:27:09:B5:04 (Oracle VirtualBox virtual NIC)  
Service Info: Hosts: 127.0.0.1, METASPLOITABLE3-UB1404; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 24.49 seconds
```

Tras el escáner podemos completar la tabla anterior con las versiones de los diferentes servicios.

| PORT     | STATE   | SERVICE       | VERSION                                      |
|----------|---------|---------------|----------------------------------------------|
| 21/tcp   | open    | ftp           | ProFTPD 1.3.5                                |
| 22/tcp   | open    | ssh           | OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0) |
| 80/tcp   | open    | http          | Apache httpd 2.4.7                           |
| 445/tcp  | open    | netbios-ssn   | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  |
| 631/tcp  | open    | ipp           | CUPS 1.7                                     |
| 3306/tcp | open    | mysql         | MySQL (unauthorized)                         |
| 8080/tcp | open    | http          | Jetty 8.1.7.v20120910                        |

## 3.3 Scripts y análisis

Ahora procederemos a usar scripts para obtener más información del sistema, en este caso empezaremos por obtener información de posibles inicios de sesión sin creedenciales. Para ello he preparado una serie de scripts que lanzaré todos juntos.

Los scripts utilizados son:
- **ftp-anon**:  Busca configuraciones de servidores FTP que permitan el acceso anónimo.
- **smb-enum-users**: Este script se utiliza para enumerar usuarios en sistemas que utilizan el protocolo SMB (Server Message Block).
- **cups-info**: Este script de Nmap recopila información sobre impresoras y servicios de impresión utilizando el protocolo CUPS (Common Unix Printing System).
- **mysql-empty-password**: Este script se centra en la detección de servidores MySQL que tienen configuradas contraseñas vacías.

``` sh
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap --script ftp-anon,smb-enum-users,cups-info,mysql-empty-password 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:14 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.0034s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
80/tcp   open   http  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
3000/tcp closed ppp  
3306/tcp open   mysql  
|_mysql-empty-password: Host '192.168.56.1' is not allowed to connect to this MySQL server  
8080/tcp open   http-proxy  
8181/tcp closed intermapper  
  
Host script results:  
| smb-enum-users:   
|   METASPLOITABLE3-UB1404\chewbacca (RID: 1000)  
|     Full name:     
|     Description:   
|_    Flags:       Normal user account  
  
Nmap done: 1 IP address (1 host up) scanned in 22.16 seconds  
```

Como vemos no hemos tenido suerte en el uso de los scripts. Se nos informa de que nuestro host no tiene permitido conectarse al servidor MySQL.

Ahora vamos a comprobar las vulnerabilidades que se presentan en nuestro equipo, en este caso se ha realizado de dos formas, con el script **vuln**, que se encuentra por defecto con la isntalación de nmap, y con el script **[vulscan](https://github.com/scipag/vulscan)**. Eso lo he hecho ya que el script **vuln** me ha dado unos resultados dudosos, ya que no fué capaz de encontrar vulnerabilidades en el servicio ftp.

A continuación tenemos extractos importantes sobre el uso de nmap con el script **vuln**:

``` sh
┌──(carlosmofe㉿reaper)-[~]  
└─$ sudo nmap --script vuln -p 21,22,80,445,631,3000,3306,8080,8181 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:17 CET  
Pre-scan script results:  
| broadcast-avahi-dos:   
|   Discovered hosts:  
|     224.0.0.251  
|   After NULL UDP avahi packet DoS (CVE-2011-1002).  
|_  Hosts are all up (not vulnerable).  
80/tcp   open   http  
| http-enum:   
|   /: Root directory w/ listing on 'apache/2.4.7 (ubuntu)'  
|   /phpmyadmin/: phpMyAdmin  
|_  /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.7 (ubuntu)'  
| http-sql-injection:   
|   Possible sqli for queries:  
|     http://192.168.56.8:80/?C=D%3BO%3DA%27%20OR%20sqlspider  ---> Posibles inyecciones SQL
|     http://192.168.56.8:80/?C=M%3BO%3DA%27%20OR%20sqlspider  
|     http://192.168.56.8:80/?C=N%3BO%3DD%27%20OR%20sqlspider  
|_    http://192.168.56.8:80/?C=S%3BO%3DA%27%20OR%20sqlspider  
| http-slowloris-check:   
|   VULNERABLE:  
|   Slowloris DOS attack  
|     State: LIKELY VULNERABLE  
|     IDs:  CVE:CVE-2007-6750  ----> Posiblemente vulnerable a una DOS.
|       Slowloris tries to keep many connections to the target web server open and hold  
|       them open as long as possible.  It accomplishes this by opening connections to  
|       the target web server and sending a partial request. By doing so, it starves  
|       the http servers resources causing Denial Of Service.  
|         
|     Disclosure date: 2009-09-17  
|     References:  
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750  
|_      http://ha.ckers.org/slowloris/  
| http-csrf:   
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.56.8  
|   Found the following possible CSRF vulnerabilities:   
|       
|     Path: http://192.168.56.8:80/drupal/  ----> Posibles vulnerabilidades de CSRF
|     Form id: user-login-form  
|     Form action: /drupal/?q=node&destination=node  
|       
|     Path: http://192.168.56.8:80/drupal/?q=user/password  
|     Form id: user-pass  
|     Form action: /drupal/?q=user/password  

|     Path: http://192.168.56.8:80/drupal/?q=node/1  
|     Form id: user-login-form  
|_    Form action: /drupal/?q=node/1&destination=node/1  
631/tcp  open   ipp  
|_http-aspnet-debug: ERROR: Script execution failed (use -d to debug)  
| http-enum:   ---> Encontró urls de administración en CUPS.
|   /admin.php: Possible admin folder  
|   /admin/: Possible admin folder  
|   /admin/admin/: Possible admin folder  
|   /administrator/: Possible admin folder  
Host script results:  
|_smb-vuln-ms10-061: false  
| smb-vuln-regsvc-dos:   
|   VULNERABLE:  ----> Vulnerable a denegaciones de servicio.
|   Service regsvc in Microsoft Windows systems vulnerable to denial of service  
|     State: VULNERABLE  
|       The service regsvc in Microsoft Windows 2000 systems is vulnerable to denial of service caused by a null deference  
|       pointer. This script will crash the service if it is vulnerable. This vulnerability was discovered by Ron Bowes  
|       while working on smb-enum-sessions.  
|_            
|_smb-vuln-ms10-054: false  
  
Nmap done: 1 IP address (1 host up) scanned in 366.46 seconds
```

Con la información que nos ha sacado este comando concluimos que:

| PORT     | SERVICE       | Vulnerabilidades                |
|----------|---------------|----------------------------------------------|
| 80/tcp   |  http          | Inyecciones SQL y denegación de servicio                       |
| 445/tcp   |  smb          | Vulnerable a denegaciones de servicio.                      |
| 631/tcp  |  ipp           | Urls de administración visibles.                                    |
| 3306/tcp |  mysql         | MySQL (unauthorized)                         |

Tras las sospechas de que este escaneo no ha sido suficiente, uso el script **vulcan**.

``` sh
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap -sV --script=vulscan/vulscan.nse 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:38 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.0020s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE     VERSION  
21/tcp   open   ftp         ProFTPD 1.3.5  
| IBM X-Force - https://exchange.xforce.ibmcloud.com:  
| [80980] ProFTPD FTP commands symlink  
| [71226] ProFTPD pool code execution  
| [65207] ProFTPD mod_sftp module denial of service  
| [64495] ProFTPD sql_prepare_where() buffer overflow  
| [63658] ProFTPD FTP server backdoor  
| [63407] mod_sql module for ProFTPD buffer overflow  
|   

22/tcp   open   ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)  
| SecurityFocus - https://www.securityfocus.com/bid/:  
| [102780] OpenSSH CVE-2016-10708 Multiple Denial of Service Vulnerabilities  
| [101552] OpenSSH 'sftp-server.c' Remote Security Bypass Vulnerability  
| [94977] OpenSSH CVE-2016-10011 Local Information Disclosure Vulnerability  
  
|   
Usernames
```

He recortado la salida del script ya que es extremadamente larga, pero como vemos ahora aparecen vulnerabilidades nuevas como pueden ser un buffer overflow para el servicio ProFTPD 1.3.5  y vulnerabilidades de denegación de servicio, bypass de seguridad y problemas de reporte de infromación local para el servicio ssh.

Por último vamos a utilizar scripts para comprobar los servicios web habilitados.
Para ello realizaremos un scaner en los puertos 80() y 443, comunmente usados para http y https respectivamente. Pero ¿Qué pasaría si un servicio web no se ha alojado en esos puertos?, pues para ello podríamos usar el script **http-enum**, el cual trae incorporado un spider (un indexador) que procuraría obtener información sobre como está indexada la web. Esto funcionaría con cualquier puerto.

(con --script=http-enum se puede hacer)
```
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap -sV -p 80,443 192.168.56.8   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:33 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.00081s latency).  
  
PORT    STATE    SERVICE VERSION  
80/tcp  open     http    Apache httpd 2.4.7 ((Ubuntu))  
443/tcp filtered https  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 20.51 seconds  
```

Para terminar, usaremos los scripts por defecto de nmap para complementar información, esto se hacer con la opción -sC.

``` sh
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap -sC 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:34 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.00041s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
| ssh-hostkey:   
|   1024 2b:2e:1f:a4:54:26:87:76:12:26:59:58:0d:da:3b:04 (DSA)  
|   2048 c9:ac:70:ef:f8:de:8b:a3:a3:44:ab:3d:32:0a:5c:6a (RSA)  
|   256 c0:49:cc:18:7b:27:a4:07:0d:2a:0d:bb:42:4c:36:17 (ECDSA)  
|_  256 a0:76:f3:76:f8:f0:70:4d:09:ca:e1:10:fd:a9:cc:0a (ED25519)  
80/tcp   open   http  
|_http-title: Index of /  
| http-ls: Volume /  
| SIZE  TIME              FILENAME  
| -     2020-10-29 19:37  chat/  
| -     2011-07-27 20:17  drupal/  
| 1.7K  2020-10-29 19:37  payroll_app.php  
| -     2013-04-08 12:06  phpmyadmin/  
|_  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
| http-robots.txt: 1 disallowed entry   
|_/  
|_ssl-date: 2023-12-13T19:35:16+00:00; -1s from scanner time.  
| ssl-cert: Subject: commonName=ubuntu  
| Not valid before: 2020-10-29T19:28:07  
|_Not valid after:  2030-10-27T19:28:07  
| http-methods:   
|_  Potentially risky methods: PUT  
|_http-title: Home - CUPS 1.7.2  
3000/tcp closed ppp  
3306/tcp open   mysql  
8080/tcp open   http-proxy  
|_http-title: Error 404 - Not Found  
8181/tcp closed intermapper  
  
Host script results:  
| smb-os-discovery:   
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)  
|   Computer name: metasploitable3-ub1404  
|   NetBIOS computer name: METASPLOITABLE3-UB1404\x00  
|   Domain name: \x00  
|   FQDN: metasploitable3-ub1404  
|_  System time: 2023-12-13T19:35:02+00:00  
| smb2-security-mode:   
|   3:1:1:   
|_    Message signing enabled but not required  
| smb2-time:   
|   date: 2023-12-13T19:35:01  
|_  start_date: N/A  
|_clock-skew: mean: 0s, deviation: 2s, median: -2s  
| smb-security-mode:   
|   account_used: guest  
|   authentication_level: user  
|   challenge_response: supported  
|_  message_signing: disabled (dangerous, but default)  
  
Nmap done: 1 IP address (1 host up) scanned in 58.34 seconds  
```

Entre la información obtenida tenemos las claves ssh del host, el conocimeinto de que en el puerto 80 tenemos estas cuatro urls:
- chat/  -> Un chat
- drupal/  -> Un CMS
- payroll_app.php  -> Un código php
- phpmyadmin/  -> Página de admin de php.

Obtenemos que en servicio alojado en el puerto 631 el método http PUT puede resultar peligroso. Y que con el script **smb-os-discovery** hemos obtenido el sistema operativo, que no concuerda con los resultados obtenidos anteriormente, OS: Windows 6.1 (Samba 4.3.11-Ubuntu). También obtenemos que la cuenta invitado en smb está habilitada.
# 4.Resumen de hallazgos clave

El informe sobre el escaneo de puertos realizado en la máquina virtual Metasploitable 3 ha revelado varias vulnerabilidades y aspectos críticos en la seguridad del dispositivo analizado. Utilizando Nmap, se identificaron múltiples puertos abiertos que exponen servicios como FTP, SSH, HTTP, SMB, IPP, MySQL y otros, algunos con configuraciones inseguras como acceso anónimo o versiones obsoletas.
## 5. Anexos

En este anexo se encuentran los diferentes comandos utilizados con las salidas para cada cuestión planteada en la práctica.

### a) Identifica la IP del equipo objetivo.
```
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap -sn 192.168.56.0/24         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 19:48 CET  
Nmap scan report for 192.168.56.1  
Host is up (0.0023s latency).  
Nmap scan report for 192.168.56.8  
Host is up (0.00092s latency).  
Nmap done: 256 IP addresses (2 hosts up) scanned in 15.94 seconds
```
### b) Identifica Sistema operativo.
```
┌──(carlosmofe㉿reaper)-[~]  
└─$ sudo nmap -O 192.168.56.8     
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 19:51 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.00039s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
80/tcp   open   http  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
3000/tcp closed ppp  
3306/tcp open   mysql  
8080/tcp open   http-proxy  
8181/tcp closed intermapper  
MAC Address: 08:00:27:09:B5:04 (Oracle VirtualBox virtual NIC)  
Aggressive OS guesses: Linux 3.2 - 4.9 (98%), Linux 3.10 - 4.11 (94%), Linux 3.13 (94%), Linux 3.13 - 3.16 (94%), OpenWrt Chaos Calmer 15.05 (Linux 3.18) or Designated Driver (Linux 4.1 or 4.4) (94%), Linux 4.10 (94%), Android 5.0 - 6.0.1 (Linux 3.4) (94%), Linux 3.10 (94%), Linux 3.2 - 3.10 (94%), Linux 3.2 - 3.16 (94%)  
No exact OS matches for host (test conditions non-ideal).  
Network Distance: 1 hop  
  
OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 21.49 seconds

c) Identifica puertos/servicios abiertos (TCP / UDP).

┌──(carlosmofe㉿reaper)-[~]  
└─$ sudo nmap -p- 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 19:54 CET  
Stats: 0:01:25 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan  
SYN Stealth Scan Timing: About 61.97% done; ETC: 19:57 (0:00:45 remaining)  
Nmap scan report for 192.168.56.8  
Host is up (0.00027s latency).  
Not shown: 65524 filtered tcp ports (no-response)  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
80/tcp   open   http  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
3000/tcp closed ppp  
3306/tcp open   mysql  
3500/tcp open   rtmp-port  
6697/tcp open   ircs-u  
8080/tcp open   http-proxy  
8181/tcp closed intermapper  
MAC Address: 08:00:27:09:B5:04 (Oracle VirtualBox virtual NIC)  
  
Nmap done: 1 IP address (1 host up) scanned in 117.10 seconds
```
### d) Identifica versiones de los servicios detectados.
```
┌──(carlosmofe㉿reaper)-[~]  
└─$ sudo nmap -sV 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 19:53 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.00028s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE     VERSION  
21/tcp   open   ftp         ProFTPD 1.3.5  
22/tcp   open   ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)  
80/tcp   open   http        Apache httpd 2.4.7  
445/tcp  open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
631/tcp  open   ipp         CUPS 1.7  
3000/tcp closed ppp  
3306/tcp open   mysql       MySQL (unauthorized)  
8080/tcp open   http        Jetty 8.1.7.v20120910  
8181/tcp closed intermapper  
MAC Address: 08:00:27:09:B5:04 (Oracle VirtualBox virtual NIC)  
Service Info: Hosts: 127.0.0.1, METASPLOITABLE3-UB1404; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 24.49 seconds
```
### e) Comprueba si existen usuarios con contraseñas vacías (NSE).
```
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap --script ftp-anon,smb-enum-users,cups-info,mysql-empty-password 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:14 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.0034s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
80/tcp   open   http  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
3000/tcp closed ppp  
3306/tcp open   mysql  
|_mysql-empty-password: Host '192.168.56.1' is not allowed to connect to this MySQL server  
8080/tcp open   http-proxy  
8181/tcp closed intermapper  
  
Host script results:  
| smb-enum-users:   
|   METASPLOITABLE3-UB1404\chewbacca (RID: 1000)  
|     Full name:     
|     Description:   
|_    Flags:       Normal user account  
  
Nmap done: 1 IP address (1 host up) scanned in 22.16 seconds  
```
### f) Comprueba las vulnerabilidades existentes en el equipo (NSE).
```
┌──(carlosmofe㉿reaper)-[~]  
└─$ sudo nmap --script vuln -p 21,22,80,445,631,3000,3306,8080,8181 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:17 CET  
Pre-scan script results:  
| broadcast-avahi-dos:   
|   Discovered hosts:  
|     224.0.0.251  
|   After NULL UDP avahi packet DoS (CVE-2011-1002).  
|_  Hosts are all up (not vulnerable).  
Stats: 0:03:49 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan  
NSE Timing: About 98.79% done; ETC: 20:21 (0:00:02 remaining)  
Nmap scan report for 192.168.56.8  
Host is up (0.00038s latency).  
  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
80/tcp   open   http  
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.  
|_http-dombased-xss: Couldn't find any DOM based XSS.  
| http-enum:   
|   /: Root directory w/ listing on 'apache/2.4.7 (ubuntu)'  
|   /phpmyadmin/: phpMyAdmin  
|_  /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.7 (ubuntu)'  
| http-sql-injection:   
|   Possible sqli for queries:  
|     http://192.168.56.8:80/?C=D%3BO%3DA%27%20OR%20sqlspider  
|     http://192.168.56.8:80/?C=M%3BO%3DA%27%20OR%20sqlspider  
|     http://192.168.56.8:80/?C=N%3BO%3DD%27%20OR%20sqlspider  
|_    http://192.168.56.8:80/?C=S%3BO%3DA%27%20OR%20sqlspider  
| http-slowloris-check:   
|   VULNERABLE:  
|   Slowloris DOS attack  
|     State: LIKELY VULNERABLE  
|     IDs:  CVE:CVE-2007-6750  
|       Slowloris tries to keep many connections to the target web server open and hold  
|       them open as long as possible.  It accomplishes this by opening connections to  
|       the target web server and sending a partial request. By doing so, it starves  
|       the http server's resources causing Denial Of Service.  
|         
|     Disclosure date: 2009-09-17  
|     References:  
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750  
|_      http://ha.ckers.org/slowloris/  
| http-csrf:   
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.56.8  
|   Found the following possible CSRF vulnerabilities:   
|       
|     Path: http://192.168.56.8:80/drupal/  
|     Form id: user-login-form  
|     Form action: /drupal/?q=node&destination=node  
|       
|     Path: http://192.168.56.8:80/payroll_app.php  
|     Form id:   
|     Form action:   
|       
|     Path: http://192.168.56.8:80/chat/  
|     Form id: name  
|     Form action: index.php  
|       
|     Path: http://192.168.56.8:80/drupal/?q=node&amp;destination=node  
|     Form id: user-login-form  
|     Form action: /drupal/?q=node&destination=node%3Famp%253Bdestination%3Dnode  
|       
|     Path: http://192.168.56.8:80/drupal/?q=node/2  
|     Form id: user-login-form  
|     Form action: /drupal/?q=node/2&destination=node/2  
|       
|     Path: http://192.168.56.8:80/drupal/?q=user/password  
|     Form id: user-pass  
|     Form action: /drupal/?q=user/password  
|       
|     Path: http://192.168.56.8:80/drupal/?q=user/register  
|     Form id: user-register-form  
|     Form action: /drupal/?q=user/register  
|       
|     Path: http://192.168.56.8:80/drupal/?q=node/1  
|     Form id: user-login-form  
|_    Form action: /drupal/?q=node/1&destination=node/1  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
|_http-aspnet-debug: ERROR: Script execution failed (use -d to debug)  
| http-enum:   
|   /admin.php: Possible admin folder  
|   /admin/: Possible admin folder  
|   /admin/admin/: Possible admin folder  
|   /administrator/: Possible admin folder  
|   /adminarea/: Possible admin folder  
|   /adminLogin/: Possible admin folder  
|   /admin_area/: Possible admin folder  
|   /administratorlogin/: Possible admin folder  
|   /admin/account.php: Possible admin folder  
|   /admin/index.php: Possible admin folder  
|   /admin/login.php: Possible admin folder  
|   /admin/admin.php: Possible admin folder  
|   /admin_area/admin.php: Possible admin folder  
|   /admin_area/login.php: Possible admin folder  
|   /admin/index.html: Possible admin folder  
|   /admin/login.html: Possible admin folder  
|   /admin/admin.html: Possible admin folder  
|   /admin_area/index.php: Possible admin folder  
|   /admin/home.php: Possible admin folder  
|   /admin_area/login.html: Possible admin folder  
|   /admin_area/index.html: Possible admin folder  
|   /admin/controlpanel.php: Possible admin folder  
|   /admincp/: Possible admin folder  
|   /admincp/index.asp: Possible admin folder  
|   /admincp/index.html: Possible admin folder  
|   /admincp/login.php: Possible admin folder  
|   /admin/account.html: Possible admin folder  
|   /adminpanel.html: Possible admin folder  
|   /admin/admin_login.html: Possible admin folder  
|   /admin_login.html: Possible admin folder  
|   /admin/cp.php: Possible admin folder  
|   /administrator/index.php: Possible admin folder  
|   /administrator/login.php: Possible admin folder  
|   /admin/admin_login.php: Possible admin folder  
|   /admin_login.php: Possible admin folder  
|   /administrator/account.php: Possible admin folder  
|   /administrator.php: Possible admin folder  
|   /admin_area/admin.html: Possible admin folder  
|   /admin/admin-login.php: Possible admin folder  
|   /admin-login.php: Possible admin folder  
|   /admin/home.html: Possible admin folder  
|   /admin/admin-login.html: Possible admin folder  
|   /admin-login.html: Possible admin folder  
|   /admincontrol.php: Possible admin folder  
|   /admin/adminLogin.html: Possible admin folder  
|   /adminLogin.html: Possible admin folder  
|   /adminarea/index.html: Possible admin folder  
|   /adminarea/admin.html: Possible admin folder  
|   /admin/controlpanel.html: Possible admin folder  
|   /admin.html: Possible admin folder  
|   /admin/cp.html: Possible admin folder  
|   /adminpanel.php: Possible admin folder  
|   /administrator/index.html: Possible admin folder  
|   /administrator/login.html: Possible admin folder  
|   /administrator/account.html: Possible admin folder  
|   /administrator.html: Possible admin folder  
|   /adminarea/login.html: Possible admin folder  
|   /admincontrol/login.html: Possible admin folder  
|   /admincontrol.html: Possible admin folder  
|   /adminLogin.php: Possible admin folder  
|   /admin/adminLogin.php: Possible admin folder  
|   /adminarea/index.php: Possible admin folder  
|   /adminarea/admin.php: Possible admin folder  
|   /adminarea/login.php: Possible admin folder  
|   /admincontrol/login.php: Possible admin folder  
|   /admin2.php: Possible admin folder  
|   /admin2/login.php: Possible admin folder  
|   /admin2/index.php: Possible admin folder  
|   /administratorlogin.php: Possible admin folder  
|   /admin/account.cfm: Possible admin folder  
|   /admin/index.cfm: Possible admin folder  
|   /admin/login.cfm: Possible admin folder  
|   /admin/admin.cfm: Possible admin folder  
|   /admin.cfm: Possible admin folder  
|   /admin/admin_login.cfm: Possible admin folder  
|   /admin_login.cfm: Possible admin folder  
|   /adminpanel.cfm: Possible admin folder  
|   /admin/controlpanel.cfm: Possible admin folder  
|   /admincontrol.cfm: Possible admin folder  
|   /admin/cp.cfm: Possible admin folder  
|   /admincp/index.cfm: Possible admin folder  
|   /admincp/login.cfm: Possible admin folder  
|   /admin_area/admin.cfm: Possible admin folder  
|   /admin_area/login.cfm: Possible admin folder  
|   /administrator/login.cfm: Possible admin folder  
|   /administratorlogin.cfm: Possible admin folder  
|   /administrator.cfm: Possible admin folder  
|   /administrator/account.cfm: Possible admin folder  
|   /adminLogin.cfm: Possible admin folder  
|   /admin2/index.cfm: Possible admin folder  
|   /admin_area/index.cfm: Possible admin folder  
|   /admin2/login.cfm: Possible admin folder  
|   /admincontrol/login.cfm: Possible admin folder  
|   /administrator/index.cfm: Possible admin folder  
|   /adminarea/login.cfm: Possible admin folder  
|   /adminarea/admin.cfm: Possible admin folder  
|   /adminarea/index.cfm: Possible admin folder  
|   /admin/adminLogin.cfm: Possible admin folder  
|   /admin-login.cfm: Possible admin folder  
|   /admin/admin-login.cfm: Possible admin folder  
|   /admin/home.cfm: Possible admin folder  
|   /admin/account.asp: Possible admin folder  
|   /admin/index.asp: Possible admin folder  
|   /admin/login.asp: Possible admin folder  
|   /admin/admin.asp: Possible admin folder  
|   /admin_area/admin.asp: Possible admin folder  
|   /admin_area/login.asp: Possible admin folder  
|   /admin_area/index.asp: Possible admin folder  
|   /admin/home.asp: Possible admin folder  
|   /admin/controlpanel.asp: Possible admin folder  
|   /admin.asp: Possible admin folder  
|   /admin/admin-login.asp: Possible admin folder  
|   /admin-login.asp: Possible admin folder  
|   /admin/cp.asp: Possible admin folder  
|   /administrator/account.asp: Possible admin folder  
|   /administrator.asp: Possible admin folder  
|   /administrator/login.asp: Possible admin folder  
|   /admincp/login.asp: Possible admin folder  
|   /admincontrol.asp: Possible admin folder  
|   /adminpanel.asp: Possible admin folder  
|   /admin/admin_login.asp: Possible admin folder  
|   /admin_login.asp: Possible admin folder  
|   /adminLogin.asp: Possible admin folder  
|   /admin/adminLogin.asp: Possible admin folder  
|   /adminarea/index.asp: Possible admin folder  
|   /adminarea/admin.asp: Possible admin folder  
|   /adminarea/login.asp: Possible admin folder  
|   /administrator/index.asp: Possible admin folder  
|   /admincontrol/login.asp: Possible admin folder  
|   /admin2.asp: Possible admin folder  
|   /admin2/login.asp: Possible admin folder  
|   /admin2/index.asp: Possible admin folder  
|   /administratorlogin.asp: Possible admin folder  
|   /admin/account.aspx: Possible admin folder  
|   /admin/index.aspx: Possible admin folder  
|   /admin/login.aspx: Possible admin folder  
|   /admin/admin.aspx: Possible admin folder  
|   /admin_area/admin.aspx: Possible admin folder  
|   /admin_area/login.aspx: Possible admin folder  
|   /admin_area/index.aspx: Possible admin folder  
|   /admin/home.aspx: Possible admin folder  
|   /admin/controlpanel.aspx: Possible admin folder  
|   /admin.aspx: Possible admin folder  
|   /admin/admin-login.aspx: Possible admin folder  
|   /admin-login.aspx: Possible admin folder  
|   /admin/cp.aspx: Possible admin folder  
|   /administrator/account.aspx: Possible admin folder  
|   /administrator.aspx: Possible admin folder  
|   /administrator/login.aspx: Possible admin folder  
|   /admincp/index.aspx: Possible admin folder  
|   /admincp/login.aspx: Possible admin folder  
|   /admincontrol.aspx: Possible admin folder  
|   /adminpanel.aspx: Possible admin folder  
|   /admin/admin_login.aspx: Possible admin folder  
|   /admin_login.aspx: Possible admin folder  
|   /adminLogin.aspx: Possible admin folder  
|   /admin/adminLogin.aspx: Possible admin folder  
|   /adminarea/index.aspx: Possible admin folder  
|   /adminarea/admin.aspx: Possible admin folder  
|   /adminarea/login.aspx: Possible admin folder  
|   /administrator/index.aspx: Possible admin folder  
|   /admincontrol/login.aspx: Possible admin folder  
|   /admin2.aspx: Possible admin folder  
|   /admin2/login.aspx: Possible admin folder  
|   /admin2/index.aspx: Possible admin folder  
|   /administratorlogin.aspx: Possible admin folder  
|   /admin/index.jsp: Possible admin folder  
|   /admin/login.jsp: Possible admin folder  
|   /admin/admin.jsp: Possible admin folder  
|   /admin_area/admin.jsp: Possible admin folder  
|   /admin_area/login.jsp: Possible admin folder  
|   /admin_area/index.jsp: Possible admin folder  
|   /admin/home.jsp: Possible admin folder  
|   /admin/controlpanel.jsp: Possible admin folder  
|   /admin.jsp: Possible admin folder  
|   /admin/admin-login.jsp: Possible admin folder  
|   /admin-login.jsp: Possible admin folder  
|   /admin/cp.jsp: Possible admin folder  
|   /administrator/account.jsp: Possible admin folder  
|   /administrator.jsp: Possible admin folder  
|   /administrator/login.jsp: Possible admin folder  
|   /admincp/index.jsp: Possible admin folder  
|   /admincp/login.jsp: Possible admin folder  
|   /admincontrol.jsp: Possible admin folder  
|   /admin/account.jsp: Possible admin folder  
|   /adminpanel.jsp: Possible admin folder  
|   /admin/admin_login.jsp: Possible admin folder  
|   /admin_login.jsp: Possible admin folder  
|   /adminLogin.jsp: Possible admin folder  
|   /admin/adminLogin.jsp: Possible admin folder  
|   /adminarea/index.jsp: Possible admin folder  
|   /adminarea/admin.jsp: Possible admin folder  
|   /adminarea/login.jsp: Possible admin folder  
|   /administrator/index.jsp: Possible admin folder  
|   /admincontrol/login.jsp: Possible admin folder  
|   /admin2.jsp: Possible admin folder  
|   /admin2/login.jsp: Possible admin folder  
|   /admin2/index.jsp: Possible admin folder  
|   /administratorlogin.jsp: Possible admin folder  
|   /admin1.php: Possible admin folder  
|   /administr8.asp: Possible admin folder  
|   /administr8.php: Possible admin folder  
|   /administr8.jsp: Possible admin folder  
|   /administr8.aspx: Possible admin folder  
|   /administr8.cfm: Possible admin folder  
|   /administr8/: Possible admin folder  
|   /administer/: Possible admin folder  
|   /administracao.php: Possible admin folder  
|   /administracao.asp: Possible admin folder  
|   /administracao.aspx: Possible admin folder  
|   /administracao.cfm: Possible admin folder  
|   /administracao.jsp: Possible admin folder  
|   /administracion.php: Possible admin folder  
|   /administracion.asp: Possible admin folder  
|   /administracion.aspx: Possible admin folder  
|   /administracion.jsp: Possible admin folder  
|   /administracion.cfm: Possible admin folder  
|   /administrators/: Possible admin folder  
|   /adminpro/: Possible admin folder  
|   /admins/: Possible admin folder  
|   /admins.cfm: Possible admin folder  
|   /admins.php: Possible admin folder  
|   /admins.jsp: Possible admin folder  
|   /admins.asp: Possible admin folder  
|   /admins.aspx: Possible admin folder  
|   /administracion-sistema/: Possible admin folder  
|   /admin108/: Possible admin folder  
|   /admin_cp.asp: Possible admin folder  
|   /admin/backup/: Possible backup  
|   /admin/download/backup.sql: Possible database backup  
|   /robots.txt: Robots file  
|   /admin/upload.php: Admin File Upload  
|   /admin/CiscoAdmin.jhtml: Cisco Collaboration Server  
|   /admin-console/: JBoss Console  
|   /admin4.nsf: Lotus Domino  
|   /admin5.nsf: Lotus Domino  
|   /admin.nsf: Lotus Domino  
|   /administrator/wp-login.php: Wordpress login page.  
|   /admin/libraries/ajaxfilemanager/ajaxfilemanager.php: Log1 CMS  
|   /admin/view/javascript/fckeditor/editor/filemanager/connectors/test.html: OpenCart/FCKeditor File upload  
|   /admin/includes/tiny_mce/plugins/tinybrowser/upload.php: CompactCMS or B-Hind CMS/FCKeditor File upload  
|   /admin/includes/FCKeditor/editor/filemanager/upload/test.html: ASP Simple Blog / FCKeditor File Upload  
|   /admin/jscript/upload.php: Lizard Cart/Remote File upload  
|   /admin/jscript/upload.html: Lizard Cart/Remote File upload  
|   /admin/jscript/upload.pl: Lizard Cart/Remote File upload  
|   /admin/jscript/upload.asp: Lizard Cart/Remote File upload  
|   /admin/environment.xml: Moodle files  
|   /classes/: Potentially interesting folder  
|   /es/: Potentially interesting folder  
|   /helpdesk/: Potentially interesting folder  
|   /help/: Potentially interesting folder  
|_  /printers/: Potentially interesting folder  
3000/tcp closed ppp  
3306/tcp open   mysql  
8080/tcp open   http-proxy  
8181/tcp closed intermapper  
MAC Address: 08:00:27:09:B5:04 (Oracle VirtualBox virtual NIC)  
  
Host script results:  
|_smb-vuln-ms10-061: false  
| smb-vuln-regsvc-dos:   
|   VULNERABLE:  
|   Service regsvc in Microsoft Windows systems vulnerable to denial of service  
|     State: VULNERABLE  
|       The service regsvc in Microsoft Windows 2000 systems is vulnerable to denial of service caused by a null deference  
|       pointer. This script will crash the service if it is vulnerable. This vulnerability was discovered by Ron Bowes  
|       while working on smb-enum-sessions.  
|_            
|_smb-vuln-ms10-054: false  
  
Nmap done: 1 IP address (1 host up) scanned in 366.46 seconds
```

Con vulnscan:

```
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap -sV --script=vulscan/vulscan.nse 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:38 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.0020s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE     VERSION  
21/tcp   open   ftp         ProFTPD 1.3.5  
| vulscan: VulDB - https://vuldb.com:  
| No findings  
|   
| MITRE CVE - https://cve.mitre.org:  
| [CVE-2012-6095] ProFTPD before 1.3.5rc1, when using the UserOwner directive, allows local users to modify the ownership of arbitrary files via a race condition and a symlink attack on the (1) MKD or (2) XMKD commands.  
| [CVE-2011-4130] Use-after-free vulnerability in the Response API in ProFTPD before 1.3.3g allows remote authenticated users to execute arbitrary code via vectors involving an error that occurs after an FTP data transfer.  
| [CVE-2011-1137] Integer overflow in the mod_sftp (aka SFTP) module in ProFTPD 1.3.3d and earlier allows remote attackers to cause a denial of service (memory consumption leading to OOM kill) via a malformed SSH message.  
| [CVE-2010-4652] Heap-based buffer overflow in the sql_prepare_where function (contrib/mod_sql.c) in ProFTPD before 1.3.3d, when mod_sql is enabled, allows remote attackers to cause a denial of service (crash) and possibly execute arbitrary code via a crafted username containing substitution tags, which are not properly handled during construction of an SQL query.  
| [CVE-2010-4221] Multiple stack-based buffer overflows in the pr_netio_telnet_gets function in netio.c in ProFTPD before 1.3.3c allow remote attackers to execute arbitrary code via vectors involving a TELNET IAC escape character to a (1) FTP or (2) FTPS server.  
| [CVE-2010-3867] Multiple directory traversal vulnerabilities in the mod_site_misc module in ProFTPD before 1.3.3c allow remote authenticated users to create directories, delete directories, create symlinks, and modify file timestamps via directory traversal sequences in a (1) SITE MKDIR, (2) SITE RMDIR, (3) SITE SYMLINK, or (4) SITE UTIME command.  
| [CVE-2009-3639] The mod_tls module in ProFTPD before 1.3.2b, and 1.3.3 before 1.3.3rc2, when the dNSNameRequired TLS option is enabled, does not properly handle a '\0' character in a domain name in the Subject Alternative Name field of an X.509 client certificate, which allows remote attackers to bypass intended client-hostname restrictions via a crafted certificate issued by a legitimate Certification Authority, a related issue to CVE-2009-2408.  
| [CVE-2009-0543] ProFTPD Server 1.3.1, with NLS support enabled, allows remote attackers to bypass SQL injection protection mechanisms via invalid, encoded multibyte characters, which are not properly handled in (1) mod_sql_mysql and (2) mod_sql_postgres.  
| [CVE-2009-0542] SQL injection vulnerability in ProFTPD Server 1.3.1 through 1.3.2rc2 allows remote attackers to execute arbitrary SQL commands via a "%" (percent) character in the username, which introduces a "'" (single quote) character during variable substitution by mod_sql.  
| [CVE-2008-7265] The pr_data_xfer function in ProFTPD before 1.3.2rc3 allows remote authenticated users to cause a denial of service (CPU consumption) via an ABOR command during a data transfer.  
| [CVE-2008-4242] ProFTPD 1.3.1 interprets long commands from an FTP client as multiple commands, which allows remote attackers to conduct cross-site request forgery (CSRF) attacks and execute arbitrary FTP commands via a long ftp:// URI that leverages an existing session from the FTP client implementation in a web browser.  
| [CVE-2006-6563] Stack-based buffer overflow in the pr_ctrls_recv_request function in ctrls.c in the mod_ctrls module in ProFTPD before 1.3.1rc1 allows local users to execute arbitrary code via a large reqarglen length value.  
| [CVE-2006-6171] ** DISPUTED **  ProFTPD 1.3.0a and earlier does not properly set the buffer size limit when CommandBufferSize is specified in the configuration file, which leads to an off-by-two buffer underflow.  NOTE: in November 2006, the role of CommandBufferSize was originally associated with CVE-2006-5815, but this was an error stemming from a vague initial disclosure.  NOTE: ProFTPD developers dispute this issue, saying that the relevant memory location is overwritten by assignment before further use within the affected function, so this is not a vulnerability.  
| [CVE-2006-6170] Buffer overflow in the tls_x509_name_oneline function in the mod_tls module, as used in ProFTPD 1.3.0a and earlier, and possibly other products, allows remote attackers to execute arbitrary code via a large data length argument, a different vulnerability than CVE-2006-5815.  
| [CVE-2006-5815] Stack-based buffer overflow in the sreplace function in ProFTPD 1.3.0 and earlier allows remote attackers, probably authenticated, to cause a denial of service and execute arbitrary code, as demonstrated by vd_proftpd.pm, a "ProFTPD remote exploit."  
| [CVE-2005-4816] Buffer overflow in mod_radius in ProFTPD before 1.3.0rc2 allows remote attackers to cause a denial of service (crash) and possibly execute arbitrary code via a long password.  
| [CVE-2005-2390] Multiple format string vulnerabilities in ProFTPD before 1.3.0rc2 allow attackers to cause a denial of service or obtain sensitive information via (1) certain inputs to the shutdown message from ftpshut, or (2) the SQLShowInfo mod_sql directive.  
| [CVE-2004-0529] The modified suexec program in cPanel, when configured for mod_php and compiled for Apache 1.3.31 and earlier without mod_phpsuexec, allows local users to execute untrusted shared scripts and gain privileges, as demonstrated using untainted scripts such as (1) proftpdvhosts or (2) addalink.cgi, a different vulnerability than CVE-2004-0490.  
|   
| SecurityFocus - https://www.securityfocus.com/bid/:  
| [50631] ProFTPD Prior To 1.3.3g Use-After-Free Remote Code Execution Vulnerability  
|   
| IBM X-Force - https://exchange.xforce.ibmcloud.com:  
| [80980] ProFTPD FTP commands symlink  
| [71226] ProFTPD pool code execution  
| [65207] ProFTPD mod_sftp module denial of service  
| [64495] ProFTPD sql_prepare_where() buffer overflow  
| [63658] ProFTPD FTP server backdoor  
| [63407] mod_sql module for ProFTPD buffer overflow  
| [63155] ProFTPD pr_data_xfer denial of service  
| [62909] ProFTPD mod_site_misc directory traversal  
| [62908] ProFTPD pr_netio_telnet_gets() buffer overflow  
| [53936] ProFTPD mod_tls SSL certificate security bypass  
| [48951] ProFTPD mod_sql username percent SQL injection  
| [48558] ProFTPD NLS support SQL injection protection bypass  
| [45274] ProFTPD URL cross-site request forgery  
| [33733] ProFTPD Auth API security bypass  
| [31461] ProFTPD mod_radius buffer overflow  
| [30906] ProFTPD Controls (mod_ctrls) module buffer overflow  
| [30554] ProFTPD mod_tls module tls_x509_name_oneline() buffer overflow  
| [30147] ProFTPD sreplace() buffer overflow  
| [21530] ProFTPD mod_sql format string attack  
| [21528] ProFTPD shutdown message format string attack  
| [19410] GProFTPD file name format string attack  
| [18453] ProFTPD SITE CHGRP command allows group ownership modification  
| [17724] ProFTPD could allow an attacker to obtain valid accounts  
| [16038] ProFTPD CIDR entry ACL bypass  
| [15387] ProFTPD off-by-one _xlate_ascii_write function buffer overflow  
| [12369] ProFTPD mod_sql SQL injection  
| [12200] ProFTPD ASCII file newline buffer overflow  
| [10932] ProFTPD long PASS command buffer overflow  
| [8332] ProFTPD mod_sqlpw stores passwords in the wtmp log file  
| [7818] ProFTPD ls &quot  
| [7816] ProFTPD file globbing denial of service  
| [7126] ProFTPD fails to resolve hostnames  
| [6433] ProFTPD format string  
| [6209] proFTPD /var symlink  
| [6208] ProFTPD contains configuration error in postinst script when running as root  
| [5801] proftpd memory leak when using SIZE or USER commands  
| [5737] ProFTPD system using mod_sqlpw unauthorized access  
|   
| Exploit-DB - https://www.exploit-db.com:  
| [20690] wu-ftpd 2.4/2.5/2.6,Trolltech ftpd 1.2,ProFTPD 1.2,BeroFTPD 1.3.4 FTP glob Expansion Vulnerability  
| [16878] ProFTPD 1.3.2rc3 - 1.3.3b Telnet IAC Buffer Overflow (FreeBSD)  
| [16852] ProFTPD 1.2 - 1.3.0 sreplace Buffer Overflow (Linux)  
| [16851] ProFTPD 1.3.2rc3 - 1.3.3b Telnet IAC Buffer Overflow (Linux)  
| [15662] ProFTPD 1.3.3c compromised source remote root Trojan  
| [10044] ProFTPd 1.3.0 mod_ctrls Local Stack Overflow (opensuse)  
| [3730] ProFTPD 1.3.0/1.3.0a (mod_ctrls) Local Overflow Exploit (exec-shield)  
| [3333] ProFTPD 1.3.0/1.3.0a (mod_ctrls support) Local Buffer Overflow Exploit 2  
| [3330] ProFTPD 1.3.0/1.3.0a (mod_ctrls support) Local Buffer Overflow Exploit  
| [2928] ProFTPD <= 1.3.0a (mod_ctrls support) Local Buffer Overflow PoC  
| [2856] ProFTPD 1.3.0 (sreplace) Remote Stack Overflow Exploit (meta)  
|   
| OpenVAS (Nessus) - http://www.openvas.org:  
| [103331] ProFTPD Prior To 1.3.3g Use-After-Free Remote Code Execution Vulnerability  
| [63497] Debian Security Advisory DSA 1730-1 (proftpd-dfsg)  
|   
| SecurityTracker - https://www.securitytracker.com:  
| [1028040] ProFTPD MKD/XMKD Race Condition Lets Local Users Gain Elevated Privileges  
| [1026321] ProFTPD Use-After-Free Memory Error Lets Remote Authenticated Users Execute Arbitrary Code  
| [1020945] ProFTPD Request Processing Bug Permits Cross-Site Request Forgery Attacks  
| [1017931] ProFTPD Auth API State Error May Let Remote Users Access the System in Certain Cases  
| [1017167] ProFTPD sreplace() Off-by-one Bug Lets Remote Users Execute Arbitrary Code  
| [1012488] ProFTPD SITE CHGRP Command Lets Remote Authenticated Users Modify File/Directory Group Ownership  
| [1011687] ProFTPd Login Timing Differences Disclose Valid User Account Names to Remote Users  
| [1009997] ProFTPD Access Control Bug With CIDR Addresses May Let Remote Authenticated Users Access Files  
| [1009297] ProFTPD _xlate_ascii_write() Off-By-One Buffer Overflows Let Remote Users Execute Arbitrary Code With Root Privileges  
| [1007794] ProFTPD ASCII Mode File Upload Buffer Overflow Lets Certain Remote Users Execute Arbitrary Code  
| [1007020] ProFTPD Input Validation Flaw When Authenticating Against Postgresql Using 'mod_sql' Lets Remote Users Gain Access  
| [1003019] ProFTPD FTP Server May Allow Local Users to Execute Code on the Server  
| [1002354] ProFTPD Reverse DNS Feature Fails to Check Forward-to-Reverse DNS Mappings  
| [1002148] ProFTPD Site and Quote Commands May Allow Remote Users to Execute Arbitrary Commands on the Server  
|   
| OSVDB - http://www.osvdb.org:  
| [89051] ProFTPD Multiple FTP Command Handling Symlink Arbitrary File Overwrite  
| [77004] ProFTPD Use-After-Free Response Pool Allocation List Parsing Remote Memory Corruption  
| [70868] ProFTPD mod_sftp Component SSH Payload DoS  
| [70782] ProFTPD contrib/mod_sql.c sql_prepare_where Function Crafted Username Handling Remote Overflow  
| [69562] ProFTPD on ftp.proftpd.org Compromised Source Packages Trojaned Distribution  
| [69200] ProFTPD pr_data_xfer Function ABOR Command Remote DoS  
| [68988] ProFTPD mod_site_misc Module Multiple Command Traversal Arbitrary File Manipulation  
| [68985] ProFTPD netio.c pr_netio_telnet_gets Function TELNET_IAC Escape Sequence Remote Overflow  
| [59292] ProFTPD mod_tls Module Certificate Authority (CA) subjectAltName Field Null Byte Handling SSL MiTM Weakness  
| [57311] ProFTPD contrib/mod_ratio.c Multiple Unspecified Buffer Handling Issues  
| [57310] ProFTPD Multiple Unspecified Overflows  
| [57309] ProFTPD src/support.c Unspecified Buffer Handling Issue  
| [57308] ProFTPD modules/mod_core.c Multiple Unspecified Overflows  
| [57307] ProFTPD Multiple Modules Unspecified Overflows  
| [57306] ProFTPD contrib/mod_pam.c Multiple Unspecified Buffer Handling Issues  
| [57305] ProFTPD src/main.c Unspecified Overflow  
| [57304] ProFTPD src/log.c Logfile Handling Unspecified Race Condition  
| [57303] ProFTPD modules/mod_auth.c Unspecified Issue  
| [51954] ProFTPD Server NLS Support mod_sql_* Encoded Multibyte Character SQL Injection Protection Bypass  
| [51953] ProFTPD Server mod_sql username % Character Handling SQL Injection  
| [51849] ProFTPD Character Encoding SQL Injection  
| [51720] ProFTPD NLST Command Argument Handling Remote Overflow  
| [51719] ProFTPD MKDIR Command Directory Name Handling Remote Overflow  
| [48411] ProFTPD FTP Command Truncation CSRF  
| [34602] ProFTPD Auth API Multiple Auth Module Authentication Bypass  
| [31509] ProFTPD mod_ctrls Module pr_ctrls_recv_request Function Local Overflow  
| [30719] mod_tls Module for ProFTPD tls_x509_name_oneline Function Remote Overflow  
| [30660] ProFTPD CommandBufferSize Option cmd_loop() Function DoS  
| [30267] ProFTPD src/support.c sreplace() Function Remote Overflow  
| [23063] ProFTPD mod_radius Password Overflow DoS  
| [20212] ProFTPD Host Reverse Resolution Failure ACL Bypass  
| [18271] ProFTPD mod_sql SQLShowInfo Directive Format String  
| [18270] ProFTPD ftpshut Shutdown Message Format String  
| [14012] GProftpd gprostats Utility Log Parser Remote Format String  
| [10769] ProFTPD File Transfer Newline Character Overflow  
| [10768] ProFTPD STAT Command Remote DoS  
| [10758] ProFTPD Login Timing Account Name Enumeration  
| [10173] ProFTPD mod_sqlpw wtmp Authentication Credential Disclosure  
| [9507] PostgreSQL Authentication Module (mod_sql) for ProFTPD USER Name Parameter SQL Injection  
| [9163] ProFTPD MKDIR Directory Creation / Change Remote Overflow (palmetto)  
| [7166] ProFTPD SIZE Command Memory Leak Remote DoS  
| [7165] ProFTPD USER Command Memory Leak DoS  
| [5744] ProFTPD CIDR IP Subnet ACL Bypass  
| [5705] ProFTPD Malformed cwd Command Format String  
| [5638] ProFTPD on Debian Linux postinst Installation Privilege Escalation  
| [4134] ProFTPD in_xlate_ascii_write() Function RETR Command Remote Overflow  
| [144] ProFTPD src/log.c log_xfer() Function Remote Overflow  
|_  
22/tcp   open   ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)  
| vulscan: VulDB - https://vuldb.com:  
| No findings  
|   
| MITRE CVE - https://cve.mitre.org:  
| [CVE-2012-5975] The SSH USERAUTH CHANGE REQUEST feature in SSH Tectia Server 6.0.4 through 6.0.20, 6.1.0 through 6.1.12, 6.2.0 through 6.2.5, and 6.3.0 through 6.3.2 on UNIX and Linux, when old-style password authentication is enabled, allows remote attackers to bypass authentication via a crafted session involving entry of blank passwords, as demonstrated by a root login session from a modified OpenSSH client with an added input_userauth_passwd_changereq call in sshconnect2.c.  
| [CVE-2012-5536] A certain Red Hat build of the pam_ssh_agent_auth module on Red Hat Enterprise Linux (RHEL) 6 and Fedora Rawhide calls the glibc error function instead of the error function in the OpenSSH codebase, which allows local users to obtain sensitive information from process memory or possibly gain privileges via crafted use of an application that relies on this module, as demonstrated by su and sudo.  
| [CVE-2010-5107] The default configuration of OpenSSH through 6.1 enforces a fixed time limit between establishing a TCP connection and completing a login, which makes it easier for remote attackers to cause a denial of service (connection-slot exhaustion) by periodically making many new TCP connections.  
| [CVE-2008-1483] OpenSSH 4.3p2, and probably other versions, allows local users to hijack forwarded X connections by causing ssh to set DISPLAY to :10, even when another process is listening on the associated port, as demonstrated by opening TCP port 6010 (IPv4) and sniffing a cookie sent by Emacs.  
| [CVE-2007-3102] Unspecified vulnerability in the linux_audit_record_event function in OpenSSH 4.3p2, as used on Fedora Core 6 and possibly other systems, allows remote attackers to write arbitrary characters to an audit log via a crafted username.  NOTE: some of these details are obtained from third party information.  
| [CVE-2004-2414] Novell NetWare 6.5 SP 1.1, when installing or upgrading using the Overlay CDs and performing a custom installation with OpenSSH, includes sensitive password information in the (1) NIOUTPUT.TXT and (2) NI.LOG log files, which might allow local users to obtain the passwords.  
|   
| SecurityFocus - https://www.securityfocus.com/bid/:  
| [102780] OpenSSH CVE-2016-10708 Multiple Denial of Service Vulnerabilities  
| [101552] OpenSSH 'sftp-server.c' Remote Security Bypass Vulnerability  
| [94977] OpenSSH CVE-2016-10011 Local Information Disclosure Vulnerability  
| [94975] OpenSSH CVE-2016-10012 Security Bypass Vulnerability  
| [94972] OpenSSH CVE-2016-10010 Privilege Escalation Vulnerability  
| [94968] OpenSSH CVE-2016-10009 Remote Code Execution Vulnerability  
| [93776] OpenSSH 'ssh/kex.c' Denial of Service Vulnerability  
| [92212] OpenSSH CVE-2016-6515 Denial of Service Vulnerability  
| [92210] OpenSSH CBC Padding Weak Encryption Security Weakness  
| [92209] OpenSSH MAC Verification Security Bypass Vulnerability  
| [91812] OpenSSH CVE-2016-6210 User Enumeration Vulnerability  
| [90440] OpenSSH CVE-2004-1653 Remote Security Vulnerability  
| [90340] OpenSSH CVE-2004-2760 Remote Security Vulnerability  
| [89385] OpenSSH CVE-2005-2666 Local Security Vulnerability  
| [88655] OpenSSH CVE-2001-1382 Remote Security Vulnerability  
| [88513] OpenSSH CVE-2000-0999 Remote Security Vulnerability  
| [88367] OpenSSH CVE-1999-1010 Local Security Vulnerability  
| [87789] OpenSSH CVE-2003-0682 Remote Security Vulnerability  
| [86187] OpenSSH 'session.c' Local Security Bypass Vulnerability  
| [86144] OpenSSH CVE-2007-2768 Remote Security Vulnerability  
| [84427] OpenSSH CVE-2016-1908 Security Bypass Vulnerability  
| [84314] OpenSSH CVE-2016-3115 Remote Command Injection Vulnerability  
| [84185] OpenSSH CVE-2006-4925 Denial-Of-Service Vulnerability  
| [81293] OpenSSH CVE-2016-1907 Denial of Service Vulnerability  
| [80698] OpenSSH CVE-2016-0778 Heap Based Buffer Overflow Vulnerability  
| [80695] OpenSSH CVE-2016-0777 Information Disclosure Vulnerability  
| [76497] OpenSSH CVE-2015-6565 Local Security Bypass Vulnerability  
| [76317] OpenSSH PAM Support Multiple Remote Code Execution Vulnerabilities  
| [75990] OpenSSH Login Handling Security Bypass Weakness  
| [75525] OpenSSH 'x11_open_helper()' Function Security Bypass Vulnerability  
| [71420] Portable OpenSSH 'gss-serv-krb5.c' Security Bypass Vulnerability  
| [68757] OpenSSH Multiple Remote Denial of Service Vulnerabilities  
| [66459] OpenSSH Certificate Validation Security Bypass Vulnerability  
| [66355] OpenSSH 'child_set_env()' Function Security Bypass Vulnerability  
| [65674] OpenSSH 'ssh-keysign.c' Local Information Disclosure Vulnerability  
| [65230] OpenSSH 'schnorr.c' Remote Memory Corruption Vulnerability  
| [63605] OpenSSH 'sshd' Process Remote Memory Corruption Vulnerability  
| [61286] OpenSSH Remote Denial of Service Vulnerability  
| [58894] GSI-OpenSSH PAM_USER Security Bypass Vulnerability  
| [58162] OpenSSH CVE-2010-5107 Denial of Service Vulnerability  
| [54114] OpenSSH 'ssh_gssapi_parse_ename()' Function Denial of Service Vulnerability  
| [51702] Debian openssh-server Forced Command Handling Information Disclosure Vulnerability  
| [50416] Linux Kernel 'kdump' and 'mkdumprd' OpenSSH Integration Remote Information Disclosure Vulnerability  
| [49473] OpenSSH Ciphersuite Specification Information Disclosure Weakness  
| [48507] OpenSSH 'pam_thread()' Remote Buffer Overflow Vulnerability  
| [47691] Portable OpenSSH 'ssh-keysign' Local Unauthorized Access Vulnerability  
| [46155] OpenSSH Legacy Certificate Signing Information Disclosure Vulnerability  
| [45304] OpenSSH J-PAKE Security Bypass Vulnerability  
| [36552] Red Hat Enterprise Linux OpenSSH 'ChrootDirectory' Option Local Privilege Escalation Vulnerability  
| [32319] OpenSSH CBC Mode Information Disclosure Vulnerability  
| [30794] Red Hat OpenSSH Backdoor Vulnerability  
| [30339] OpenSSH 'X11UseLocalhost' X11 Forwarding Session Hijacking Vulnerability  
| [30276] Debian OpenSSH SELinux Privilege Escalation Vulnerability  
| [28531] OpenSSH ForceCommand Command Execution Weakness  
| [28444] OpenSSH X Connections Session Hijacking Vulnerability  
| [26097] OpenSSH LINUX_AUDIT_RECORD_EVENT Remote Log Injection Weakness  
| [25628] OpenSSH X11 Cookie Local Authentication Bypass Vulnerability  
| [23601] OpenSSH S/Key Remote Information Disclosure Vulnerability  
| [20956] OpenSSH Privilege Separation Key Signature Weakness  
| [20418] OpenSSH-Portable Existing Password Remote Information Disclosure Weakness  
| [20245] OpenSSH-Portable GSSAPI Authentication Abort Information Disclosure Weakness  
| [20241] Portable OpenSSH GSSAPI Remote Code Execution Vulnerability  
| [20216] OpenSSH Duplicated Block Remote Denial of Service Vulnerability  
| [16892] OpenSSH Remote PAM Denial Of Service Vulnerability  
| [14963] OpenSSH LoginGraceTime Remote Denial Of Service Vulnerability  
| [14729] OpenSSH GSSAPI Credential Disclosure Vulnerability  
| [14727] OpenSSH DynamicForward Inadvertent GatewayPorts Activation Vulnerability  
| [11781] OpenSSH-portable PAM Authentication Remote Information Disclosure Vulnerability  
| [9986] RCP, OpenSSH SCP Client File Corruption Vulnerability  
| [9040] OpenSSH PAM Conversation Memory Scrubbing Weakness  
| [8677] Multiple Portable OpenSSH PAM Vulnerabilities  
| [8628] OpenSSH Buffer Mismanagement Vulnerabilities  
| [7831] OpenSSH Reverse DNS Lookup Access Control Bypass Vulnerability  
| [7482] OpenSSH Remote Root Authentication Timing Side-Channel Weakness  
| [7467] OpenSSH-portable Enabled PAM Delay Information Disclosure Vulnerability  
| [7343] OpenSSH Authentication Execution Path Timing Information Leakage Weakness  
| [6168] OpenSSH Visible Password Vulnerability  
| [5374] OpenSSH Trojan Horse Vulnerability  
| [5093] OpenSSH Challenge-Response Buffer Overflow Vulnerabilities  
| [4560] OpenSSH Kerberos 4 TGT/AFS Token Buffer Overflow Vulnerability  
| [4241] OpenSSH Channel Code Off-By-One Vulnerability  
| [3614] OpenSSH UseLogin Environment Variable Passing Vulnerability  
| [3560] OpenSSH Kerberos Arbitrary Privilege Elevation Vulnerability  
| [3369] OpenSSH Key Based Source IP Access Control Bypass Vulnerability  
| [3345] OpenSSH SFTP Command Restriction Bypassing Vulnerability  
| [2917] OpenSSH PAM Session Evasion Vulnerability  
| [2825] OpenSSH Client X11 Forwarding Cookie Removal File Symbolic Link Vulnerability  
| [2356] OpenSSH Private Key Authentication Check Vulnerability  
| [1949] OpenSSH Client Unauthorized Remote Forwarding Vulnerability  
| [1334] OpenSSH UseLogin Vulnerability  
|   
| IBM X-Force - https://exchange.xforce.ibmcloud.com:  
| [83258] GSI-OpenSSH auth-pam.c security bypass  
| [82781] OpenSSH time limit denial of service  
| [82231] OpenSSH pam_ssh_agent_auth PAM code execution  
| [74809] OpenSSH ssh_gssapi_parse_ename denial of service  
| [72756] Debian openssh-server commands information disclosure  
| [68339] OpenSSH pam_thread buffer overflow  
| [67264] OpenSSH ssh-keysign unauthorized access  
| [65910] OpenSSH remote_glob function denial of service  
| [65163] OpenSSH certificate information disclosure  
| [64387] OpenSSH J-PAKE security bypass  
| [63337] Cisco Unified Videoconferencing OpenSSH weak security  
| [46620] OpenSSH and multiple SSH Tectia products CBC mode information disclosure  
| [45202] OpenSSH signal handler denial of service  
| [44747] RHEL OpenSSH backdoor  
| [44280] OpenSSH PermitRootLogin information disclosure  
| [44279] OpenSSH sshd weak security  
| [44037] OpenSSH sshd SELinux role unauthorized access  
| [43940] OpenSSH X11 forwarding information disclosure  
| [41549] OpenSSH ForceCommand directive security bypass  
| [41438] OpenSSH sshd session hijacking  
| [40897] OpenSSH known_hosts weak security  
| [40587] OpenSSH username weak security  
| [37371] OpenSSH username data manipulation  
| [37118] RHSA update for OpenSSH privilege separation monitor authentication verification weakness not installed  
| [37112] RHSA update for OpenSSH signal handler race condition not installed  
| [37107] RHSA update for OpenSSH identical block denial of service not installed  
| [36637] OpenSSH X11 cookie privilege escalation  
| [35167] OpenSSH packet.c newkeys[mode] denial of service  
| [34490] OpenSSH OPIE information disclosure  
| [33794] OpenSSH ChallengeResponseAuthentication information disclosure  
| [32975] Apple Mac OS X OpenSSH denial of service  
| [32387] RHSA-2006:0738 updates for openssh not installed  
| [32359] RHSA-2006:0697 updates for openssh not installed  
| [32230] RHSA-2006:0298 updates for openssh not installed  
| [32132] RHSA-2006:0044 updates for openssh not installed  
| [30120] OpenSSH privilege separation monitor authentication verification weakness  
| [29255] OpenSSH GSSAPI user enumeration  
| [29254] OpenSSH signal handler race condition  
| [29158] OpenSSH identical block denial of service  
| [28147] Apple Mac OS X OpenSSH nonexistent user login denial of service  
| [25116] OpenSSH OpenPAM denial of service  
| [24305] OpenSSH SCP shell expansion command execution  
| [22665] RHSA-2005:106 updates for openssh not installed  
| [22117] OpenSSH GSSAPI allows elevated privileges  
| [22115] OpenSSH GatewayPorts security bypass  
| [20930] OpenSSH sshd.c LoginGraceTime denial of service  
| [19441] Sun Solaris OpenSSH LDAP (1) client authentication denial of service  
| [17213] OpenSSH allows port bouncing attacks  
| [16323] OpenSSH scp file overwrite  
| [13797] OpenSSH PAM information leak  
| [13271] OpenSSH could allow an attacker to corrupt the PAM conversion stack  
| [13264] OpenSSH PAM code could allow an attacker to gain access  
| [13215] OpenSSH buffer management errors could allow an attacker to execute code  
| [13214] OpenSSH memory vulnerabilities  
| [13191] OpenSSH large packet buffer overflow  
| [12196] OpenSSH could allow an attacker to bypass login restrictions  
| [11970] OpenSSH could allow an attacker to obtain valid administrative account  
| [11902] OpenSSH PAM support enabled information leak  
| [9803] OpenSSH &quot  
| [9763] OpenSSH downloaded from the OpenBSD FTP site or OpenBSD FTP mirror sites could contain a Trojan Horse  
| [9307] OpenSSH is running on the system  
| [9169] OpenSSH &quot  
| [8896] OpenSSH Kerberos 4 TGT/AFS buffer overflow  
| [8697] FreeBSD libutil in OpenSSH fails to drop privileges prior to using the login class capability database  
| [8383] OpenSSH off-by-one error in channel code  
| [7647] OpenSSH UseLogin option arbitrary code execution  
| [7634] OpenSSH using sftp and restricted keypairs could allow an attacker to bypass restrictions  
| [7598] OpenSSH with Kerberos allows attacker to gain elevated privileges  
| [7179] OpenSSH source IP access control bypass  
| [6757] OpenSSH &quot  
| [6676] OpenSSH X11 forwarding symlink attack could allow deletion of arbitrary files  
| [6084] OpenSSH 2.3.1 allows remote users to bypass authentication  
| [5517] OpenSSH allows unauthorized access to resources  
| [4646] OpenSSH UseLogin option allows remote users to execute commands as root  
|   
| Exploit-DB - https://www.exploit-db.com:  
| [14866] Novell Netware 6.5 - OpenSSH Remote Stack Overflow  
|   
| OpenVAS (Nessus) - http://www.openvas.org:  
| [902488] OpenSSH 'sshd' GSSAPI Credential Disclosure Vulnerability  
| [900179] OpenSSH CBC Mode Information Disclosure Vulnerability  
| [881183] CentOS Update for openssh CESA-2012:0884 centos6   
| [880802] CentOS Update for openssh CESA-2009:1287 centos5 i386  
| [880746] CentOS Update for openssh CESA-2009:1470 centos5 i386  
| [870763] RedHat Update for openssh RHSA-2012:0884-04  
| [870129] RedHat Update for openssh RHSA-2008:0855-01  
| [861813] Fedora Update for openssh FEDORA-2010-5429  
| [861319] Fedora Update for openssh FEDORA-2007-395  
| [861170] Fedora Update for openssh FEDORA-2007-394  
| [861012] Fedora Update for openssh FEDORA-2007-715  
| [840345] Ubuntu Update for openssh vulnerability USN-597-1  
| [840300] Ubuntu Update for openssh update USN-612-5  
| [840271] Ubuntu Update for openssh vulnerability USN-612-2  
| [840268] Ubuntu Update for openssh update USN-612-7  
| [840259] Ubuntu Update for openssh vulnerabilities USN-649-1  
| [840214] Ubuntu Update for openssh vulnerability USN-566-1  
| [831074] Mandriva Update for openssh MDVA-2010:162 (openssh)  
| [830929] Mandriva Update for openssh MDVA-2010:090 (openssh)  
| [830807] Mandriva Update for openssh MDVA-2010:026 (openssh)  
| [830603] Mandriva Update for openssh MDVSA-2008:098 (openssh)  
| [830523] Mandriva Update for openssh MDVSA-2008:078 (openssh)  
| [830317] Mandriva Update for openssh-askpass-qt MDKA-2007:127 (openssh-askpass-qt)  
| [830191] Mandriva Update for openssh MDKSA-2007:236 (openssh)  
| [802407] OpenSSH 'sshd' Challenge Response Authentication Buffer Overflow Vulnerability  
| [103503] openssh-server Forced Command Handling Information Disclosure Vulnerability  
| [103247] OpenSSH Ciphersuite Specification Information Disclosure Weakness  
| [103064] OpenSSH Legacy Certificate Signing Information Disclosure Vulnerability  
| [100584] OpenSSH X Connections Session Hijacking Vulnerability  
| [100153] OpenSSH CBC Mode Information Disclosure Vulnerability  
| [66170] CentOS Security Advisory CESA-2009:1470 (openssh)  
| [65987] SLES10: Security update for OpenSSH  
| [65819] SLES10: Security update for OpenSSH  
| [65514] SLES9: Security update for OpenSSH  
| [65513] SLES9: Security update for OpenSSH  
| [65334] SLES9: Security update for OpenSSH  
| [65248] SLES9: Security update for OpenSSH  
| [65218] SLES9: Security update for OpenSSH  
| [65169] SLES9: Security update for openssh,openssh-askpass  
| [65126] SLES9: Security update for OpenSSH  
| [65019] SLES9: Security update for OpenSSH  
| [65015] SLES9: Security update for OpenSSH  
| [64931] CentOS Security Advisory CESA-2009:1287 (openssh)  
| [61639] Debian Security Advisory DSA 1638-1 (openssh)  
| [61030] Debian Security Advisory DSA 1576-2 (openssh)  
| [61029] Debian Security Advisory DSA 1576-1 (openssh)  
| [60840] FreeBSD Security Advisory (FreeBSD-SA-08:05.openssh.asc)  
| [60803] Gentoo Security Advisory GLSA 200804-03 (openssh)  
| [60667] Slackware Advisory SSA:2008-095-01 openssh   
| [59014] Slackware Advisory SSA:2007-255-01 openssh   
| [58741] Gentoo Security Advisory GLSA 200711-02 (openssh)  
| [57919] Gentoo Security Advisory GLSA 200611-06 (openssh)  
| [57895] Gentoo Security Advisory GLSA 200609-17 (openssh)  
| [57585] Debian Security Advisory DSA 1212-1 (openssh (1:3.8.1p1-8.sarge.6))  
| [57492] Slackware Advisory SSA:2006-272-02 openssh   
| [57483] Debian Security Advisory DSA 1189-1 (openssh-krb5)  
| [57476] FreeBSD Security Advisory (FreeBSD-SA-06:22.openssh.asc)  
| [57470] FreeBSD Ports: openssh  
| [56352] FreeBSD Security Advisory (FreeBSD-SA-06:09.openssh.asc)  
| [56330] Gentoo Security Advisory GLSA 200602-11 (OpenSSH)  
| [56294] Slackware Advisory SSA:2006-045-06 openssh   
| [53964] Slackware Advisory SSA:2003-266-01 New OpenSSH packages   
| [53885] Slackware Advisory SSA:2003-259-01 OpenSSH Security Advisory   
| [53884] Slackware Advisory SSA:2003-260-01 OpenSSH updated again   
| [53788] Debian Security Advisory DSA 025-1 (openssh)  
| [52638] FreeBSD Security Advisory (FreeBSD-SA-03:15.openssh.asc)  
| [52635] FreeBSD Security Advisory (FreeBSD-SA-03:12.openssh.asc)  
| [11343] OpenSSH Client Unauthorized Remote Forwarding  
| [10954] OpenSSH AFS/Kerberos ticket/token passing  
| [10883] OpenSSH Channel Code Off by 1  
| [10823] OpenSSH UseLogin Environment Variables  
|   
| SecurityTracker - https://www.securitytracker.com:  
| [1028187] OpenSSH pam_ssh_agent_auth Module on Red Hat Enterprise Linux Lets Remote Users Execute Arbitrary Code  
| [1026593] OpenSSH Lets Remote Authenticated Users Obtain Potentially Sensitive Information  
| [1025739] OpenSSH on FreeBSD Has Buffer Overflow in pam_thread() That Lets Remote Users Execute Arbitrary Code  
| [1025482] OpenSSH ssh-keysign Utility Lets Local Users Gain Elevated Privileges  
| [1025028] OpenSSH Legacy Certificates May Disclose Stack Contents to Remote Users  
| [1022967] OpenSSH on Red Hat Enterprise Linux Lets Remote Authenticated Users Gain Elevated Privileges  
| [1021235] OpenSSH CBC Mode Error Handling May Let Certain Remote Users Obtain Plain Text in Certain Cases  
| [1020891] OpenSSH on Debian Lets Remote Users Prevent Logins  
| [1020730] OpenSSH for Red Hat Enterprise Linux Packages May Have Been Compromised  
| [1020537] OpenSSH on HP-UX Lets Local Users Hijack X11 Sessions  
| [1019733] OpenSSH Unsafe Default Configuration May Let Local Users Execute Arbitrary Commands  
| [1019707] OpenSSH Lets Local Users Hijack Forwarded X Sessions in Certain Cases  
| [1017756] Apple OpenSSH Key Generation Process Lets Remote Users Deny Service  
| [1017183] OpenSSH Privilege Separation Monitor Validation Error May Cause the Monitor to Fail to Properly Control the Unprivileged Process  
| [1016940] OpenSSH Race Condition in Signal Handler Lets Remote Users Deny Service and May Potentially Permit Code Execution  
| [1016939] OpenSSH GSSAPI Authentication Abort Error Lets Remote Users Determine Valid Usernames
```
### g) Comprueba si dispone de servicios web habilitados (NSE).

(con --script=http-enum se puede hacer)
```
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap -sV -p 80,443 192.168.56.8   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:33 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.00081s latency).  
  
PORT    STATE    SERVICE VERSION  
80/tcp  open     http    Apache httpd 2.4.7 ((Ubuntu))  
443/tcp filtered https  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 20.51 seconds  
```
### h) Ejecuta los scripts por defecto de nmap para ampliar la información (NSE).
```
┌──(carlosmofe㉿reaper)-[~]  
└─$ nmap -sC 192.168.56.8  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-13 20:34 CET  
Nmap scan report for 192.168.56.8  
Host is up (0.00041s latency).  
Not shown: 991 filtered tcp ports (no-response)  
PORT     STATE  SERVICE  
21/tcp   open   ftp  
22/tcp   open   ssh  
| ssh-hostkey:   
|   1024 2b:2e:1f:a4:54:26:87:76:12:26:59:58:0d:da:3b:04 (DSA)  
|   2048 c9:ac:70:ef:f8:de:8b:a3:a3:44:ab:3d:32:0a:5c:6a (RSA)  
|   256 c0:49:cc:18:7b:27:a4:07:0d:2a:0d:bb:42:4c:36:17 (ECDSA)  
|_  256 a0:76:f3:76:f8:f0:70:4d:09:ca:e1:10:fd:a9:cc:0a (ED25519)  
80/tcp   open   http  
|_http-title: Index of /  
| http-ls: Volume /  
| SIZE  TIME              FILENAME  
| -     2020-10-29 19:37  chat/  
| -     2011-07-27 20:17  drupal/  
| 1.7K  2020-10-29 19:37  payroll_app.php  
| -     2013-04-08 12:06  phpmyadmin/  
|_  
445/tcp  open   microsoft-ds  
631/tcp  open   ipp  
| http-robots.txt: 1 disallowed entry   
|_/  
|_ssl-date: 2023-12-13T19:35:16+00:00; -1s from scanner time.  
| ssl-cert: Subject: commonName=ubuntu  
| Not valid before: 2020-10-29T19:28:07  
|_Not valid after:  2030-10-27T19:28:07  
| http-methods:   
|_  Potentially risky methods: PUT  
|_http-title: Home - CUPS 1.7.2  
3000/tcp closed ppp  
3306/tcp open   mysql  
8080/tcp open   http-proxy  
|_http-title: Error 404 - Not Found  
8181/tcp closed intermapper  
  
Host script results:  
| smb-os-discovery:   
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)  
|   Computer name: metasploitable3-ub1404  
|   NetBIOS computer name: METASPLOITABLE3-UB1404\x00  
|   Domain name: \x00  
|   FQDN: metasploitable3-ub1404  
|_  System time: 2023-12-13T19:35:02+00:00  
| smb2-security-mode:   
|   3:1:1:   
|_    Message signing enabled but not required  
| smb2-time:   
|   date: 2023-12-13T19:35:01  
|_  start_date: N/A  
|_clock-skew: mean: 0s, deviation: 2s, median: -2s  
| smb-security-mode:   
|   account_used: guest  
|   authentication_level: user  
|   challenge_response: supported  
|_  message_signing: disabled (dangerous, but default)  
  
Nmap done: 1 IP address (1 host up) scanned in 58.34 seconds  
```