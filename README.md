# ABC de Instalación y Configuración de Servidor de Correo en Ubuntu Server

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

**Alternativa:** Ejemplo de como desplegar en un contenedor LXD (Leer documentación en https://canonical.com/lxd):
```bash
lxc launch ubuntu:20.04 servidor-correo
```
---
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
Editamos el archivo `/etc/hosts` para agregar nuestro dominio local:
```bash
sudo nano /etc/hosts
```
Esto permite que el sistema reconozca el dominio y lo resuelva internamente.

### **C. Actualización del sistema**
Es importante mantener el sistema actualizado para evitar vulnerabilidades:
```bash
sudo apt update && sudo apt upgrade
```

### **D. Instalación de Apache**
Apache es el servidor web necesario para que SquirrelMail funcione correctamente:
```bash
sudo apt-get install apache2
```
Una vez instalado, podemos probarlo accediendo desde un navegador con la IP del servidor.

### **E. Instalación de PHP y MySQL**
SquirrelMail requiere una versión antigua de PHP para su compatibilidad, por lo que añadimos un repositorio antiguo:
```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php7.4 libapache2-mod-php7.4 php-mysql
```
Para verificar la instalación de PHP:
```bash
php -v
```

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

---
## **3. Acceso desde la Red Local**
Para que otros dispositivos en la red puedan acceder al servidor:
1. Configurar el firewall para permitir tráfico en los puertos SMTP, IMAP y HTTP.
2. Verificar la dirección IP del servidor con `ip a`.
3. Probar el acceso desde un navegador con `http://[IP_DEL_SERVIDOR]/squirrelmail/`.

---
## **Mejoras Sugeridas**
  ✅ **Automatización:** Usar **Ansible** o scripts Bash para simplificar la instalación.
✅ **Seguridad:** Implementar SSL/TLS y autenticación de dos factores para mayor protección.
  ✅ **Alternativas Modernas:** Considerar **Roundcube** en lugar de SquirrelMail para una mejor experiencia de usuario.
  ✅ **Monitoreo:** Configurar herramientas como **Prometheus** o **Grafana** para supervisión.
  ✅ **Implementación en Contenedores:** Utilizar **LXD** o **Docker** para una infraestructura más flexible.

---
📌 **¡Listo! Tu servidor de correo en Ubuntu está funcionando. 🚀**
