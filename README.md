# Instalación y Configuración basica de un Servidor de Correo corporativo en Ubuntu Server

## **1. Instalación de Ubuntu Server en VMware o Contenedores LXD**
### **Pasos:**
1. Descargar e instalar **VMware Workstation** o **VMware Player**.
2. Crear una nueva máquina virtual.
3. Seleccionar la imagen ISO de **Ubuntu Server**.
4. Seguir el asistente de instalación.

**Mejoras:**
- Usar **VirtualBox** como alternativa gratuita.
- Automatizar la instalación con **Kickstart** o **Cloud-Init**.
- Implementar en contenedores **LXD** para mayor eficiencia.

### **Instalación en contenedores LXD**  
Si en lugar de una máquina virtual prefieres usar **contenedores LXD**, puedes seguir estos pasos. Para una guía detallada sobre cómo desplegar contenedores usando **LXD**, visita la siguiente página oficial de Ubuntu:  
👉 [https://documentation.ubuntu.com/lxd/en/latest/tutorial/first_steps/](https://documentation.ubuntu.com/lxd/en/latest/tutorial/first_steps/)  

Antes de crear el contenedor, puedes listar las versiones disponibles de Ubuntu para tu arquitectura con:  
```bash
lxc image list ubuntu: 24.04 architecture=$(uname -m)
```  
Luego, para crear un contenedor con la versión más reciente de Ubuntu Server:  
```bash
lxc launch ubuntu:24.04 servidor-correo
```  
También es recomendable configurar el contenedor para permitir servicios de correo ejecutando:  
```bash
lxc config set servidor-correo security.nesting true
lxc config set servidor-correo security.privileged true
```  
Para ingresar al contenedor y proceder con la instalación del servidor de correo:  
```bash
lxc exec servidor-correo -- bash
```

**Nota adicional:**

Si usas maquina virutal y deseas obtener los comandos más fácilmente, puedes clonar el repositorio. Si aún no tienes Git instalado, sigue estos pasos:

1. **Instalar Git:**
   ```bash
   sudo apt-get install git
   ```

2. **Clonar el repositorio:**
   ```bash
   git clone https://github.com/JoseDavila24/servidor-correo-ubuntu.git
   ```

3. **Ingresar al repositorio:**
   ```bash
   cd servidor-correo-ubuntu
   ```

4. **Ver el archivo README:**
   ```bash
   cat README
   ```

Esto te permitirá acceder fácilmente al repositorio y consultar el archivo `README`.

## **2. Configuración de Postfix y SquirrelMail**
### **A. Verificación de herramientas de red**
Antes de comenzar, verificamos si el sistema tiene herramientas básicas de red ejecutando un ping a `google.com`:
```bash
ping -c 4 google.com
```
Si el comando falla, es probable que falten algunas herramientas de red. Para instalarlas, ejecutamos:
```bash
sudo apt-get install -y iputils-ping iproute2
```

### **B. Configuración del dominio local**  
Para permitir que el sistema reconozca y resuelva internamente el dominio local, debes editar el archivo `/etc/hosts`. Puedes abrirlo con un editor de texto, ya sea `nano` o `vim`, según tu preferencia:  

Con **nano**:  
```bash
sudo nano /etc/hosts
```  
Con **vim**:  
```bash
sudo vim /etc/hosts
```  

Si no tienes instalado **nano** o **vim**, puedes instalarlos con el siguiente comando:  
```bash
sudo apt install -y nano vim
```  

Luego, agrega una línea similar a la siguiente, donde `servidor-correo.local` es el nombre de tu dominio local y `127.0.0.1` es la dirección de loopback de tu servidor:  
```bash
127.0.0.1   localhost  
127.0.1.1   servidor-correo.local servidor-correo  
```  

Si usaste **nano**, guarda y cierra el archivo presionando `CTRL + X`, luego confirma con `Y` y presiona `Enter`.  
Si usaste **vim**, guarda y cierra el archivo presionando `ESC`, luego escribe `:wq` y presiona `Enter`.  

Esto permitirá que tu servidor reconozca `servidor-correo.local` como su nombre de dominio local. Asegúrate de sustituir `servidor-correo.local` por el nombre de dominio que vayas a utilizar en tu red interna.

### **C. Actualización del sistema**
Es importante mantener el sistema actualizado para evitar vulnerabilidades:
```bash
sudo apt update && sudo apt upgrade
```

### **D. Instalación de Apache**  
Apache es el servidor web necesario para que SquirrelMail funcione correctamente. Para instalarlo, ejecuta:  
```bash
sudo apt-get install apache2
```  

Una vez instalado, puedes verificar que Apache está funcionando accediendo desde un navegador con la IP del servidor. Si no conoces la IP de tu servidor, puedes obtenerla con el siguiente comando:  
```bash
ip a | grep inet
```  
Esto mostrará una lista de direcciones IP. Busca la que corresponda a tu red local, por ejemplo, algo como `192.168.x.x` o `10.x.x.x`.  

Luego, abre un navegador en cualquier equipo de la misma red e ingresa:  
```
http://[IP_DEL_SERVIDOR]/
```  
Si Apache está funcionando correctamente, verás la página de inicio predeterminada de Apache.
### **E. Instalación de PHP y MySQL**
SquirrelMail requiere una versión antigua de PHP para su compatibilidad, por lo que añadimos un repositorio antiguo:
```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php7.4 libapache2-mod-php7.4 php-mysql
```
Al ejecutar el comando `php -v` para verificar la instalación de PHP, deberías ver una salida similar a la siguiente, que indica la versión de PHP que tienes instalada:

```bash
PHP 7.4.33 (cli) (built: Dec 24 2024 07:12:16) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.33, Copyright (c), by Zend Technologies
```

En este caso, muestra la versión **PHP 7.4.3** (que es la que hemos instalado). Si ves una versión diferente, es posible que tengas otra versión de PHP instalada en tu sistema.

### **F. Instalación de Postfix**
Postfix es el servidor SMTP que se encargará de enviar los correos:
```bash
sudo apt install postfix
```
Durante la instalación, seleccionamos "Internet Site" y establecemos el nombre de dominio.
Si es necesario reconfigurar Postfix, ejecutamos:
```bash
sudo dpkg-reconfigure postfix
```

### **G. Instalación de Dovecot**
Dovecot es el servidor IMAP/POP3 que nos permitirá recibir correos:
```bash
sudo apt install dovecot-imapd dovecot-pop3d
```
Reiniciamos el servicio para aplicar cambios:
```bash
sudo service dovecot restart
```

### **H. Instalación de SquirrelMail**
SquirrelMail no está en los repositorios oficiales de Ubuntu, por lo que debemos descargarlo manualmente:
```bash
cd /var/www/html/
sudo wget https://sourceforge.net/projects/squirrelmail/files/stable/1.4.22/squirrelmail-webmail-1.4.22.zip
sudo unzip squirrelmail-webmail-1.4.22.zip
```
Si no tenemos `unzip`, lo instalamos:
```bash
sudo apt install unzip
```
Renombramos la carpeta para facilitar el acceso:
```bash
sudo mv squirrelmail-webmail-1.4.22 squirrelmail
```
Asignamos los permisos correctos:
```bash
sudo chown -R www-data:www-data /var/www/html/squirrelmail/
sudo chmod 755 -R /var/www/html/squirrelmail/
```

### **I. Configuración de SquirrelMail**
Ejecutamos el asistente de configuración:
```bash
sudo perl /var/www/html/squirrelmail/config/conf.pl
```
Dentro de la configuración:
1. Seleccionamos **Opción 2**, luego **Opción 1**, y establecemos el dominio.
2. Regresamos con `R`, seleccionamos **Opción 4** y configuramos:
   - 1: `/var/www/html/squirrelmail/data/`
   - 2: `/var/www/html/squirrelmail/attach/`
   - 11: `true`
3. Guardamos con `S` y salimos con `Q`.

### **J. Creación de usuarios**
Creamos usuarios para acceder al correo:
```bash
sudo adduser usuario1
```
Esto generará un usuario con su respectiva carpeta en el servidor.

Para acceder a la interfaz de SquirrelMail desde tu navegador, una vez que hayas completado la instalación y configuración, debes ingresar la siguiente URL en tu navegador:

```
http://[IP_DEL_SERVIDOR]/squirrelmail/
```

Donde `[IP_DEL_SERVIDOR]` es la dirección IP de tu servidor de correo. Si estás trabajando en un entorno local, puedes usar `localhost` o `127.0.0.1` en lugar de la dirección IP:

```
http://localhost/squirrelmail/
```

Esto te llevará a la página de inicio de sesión de SquirrelMail, donde podrás acceder a tu bandeja de entrada y gestionar tus correos electrónicos.

---
## **3. Acceso desde la Red Local**
Para que otros dispositivos en la red puedan acceder al servidor:
1. Configurar el firewall para permitir tráfico en los puertos SMTP, IMAP y HTTP.
2. Verificar la dirección IP del servidor con `ip a`.
3. Probar el acceso desde un navegador con `http://[IP_DEL_SERVIDOR]/squirrelmail/`.

---

## **Mejoras Sugeridas**

- ✅ **Automatización:** Usar *Ansible* o scripts Bash para simplificar la instalación.  
- ✅ **Seguridad:** Implementar SSL/TLS y autenticación de dos factores para mayor protección.  
- ✅ **Alternativas Modernas:** Considerar *Roundcube* en lugar de SquirrelMail para una mejor experiencia de usuario.  
- ✅ **Monitoreo:** Configurar herramientas como *Prometheus* o *Grafana* para supervisión.  
- ✅ **Implementación en Contenedores:** Utilizar *LXD* o *Docker* para una infraestructura más flexible.  

---

📌 **¡Listo! Tu servidor de correo en Ubuntu está funcionando. 🚀**

---



**Nota del autor :D:** El archivo de configuración de Neofetch se encuentra en `~/.config/neofetch`.

