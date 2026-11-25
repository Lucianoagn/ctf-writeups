# HTB — Meow

**Plataforma:** HackTheBox  
**Dificultad:** Very Easy  
**OS:** Linux  
**Fecha:** 27/09/2026  
**Tags:** Telnet, Default Credentials

---

## Resumen

Máquina Linux muy básica que expone el servicio Telnet en el puerto 23 
con credenciales por defecto. Sin contraseña para el usuario root.

---

## Reconocimiento

### Nmap
```bash
nmap -sV -Pn -p- 10.129.x.x
```

**Resultado:**

Puerto 23 (Telnet) abierto — servicio de acceso remoto antiguo e inseguro 
que transmite todo en texto plano, sin cifrado.

---

## Explotación

Telnet no requiere cifrado ni autenticación moderna. Probé credenciales 
por defecto:

```bash
telnet 10.129.x.x
# Usuario: root
# Contraseña: (vacía)
```

Acceso directo como **root** sin contraseña.

---

## Flag

```bash
cat /root/flag.txt
```

`flag{...}`

---

## Lecciones aprendidas

- Telnet es inseguro y no debería usarse — SSH es el reemplazo correcto
- Las credenciales por defecto son un vector de ataque crítico
- Siempre cambiar contraseñas default al configurar servicios

---

## Referencias

- [Telnet vs SSH](https://www.ssh.com/academy/ssh/telnet)
