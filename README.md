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
Comprobar si se puede hacer ping a `google.com`:
```bash
ping -c 4 google.com
```
Si el comando falla, instalar las herramientas necesarias:
```bash
sudo apt-get install -y iputils-ping iproute2
```

### **B. Configuración del dominio local**
```bash
sudo nano /etc/hosts
```
*Agrega el dominio local para que el sistema lo reconozca.*
```bash
127.0.0.1  tso.localhost
```

### **C. Actualización del sistema**
```bash
sudo apt update && sudo apt upgrade
```
*Mantiene el sistema actualizado.*

### **D. Instalación de Apache**
```bash
sudo apt-get install apache2
```
*Servidor web necesario para SquirrelMail.*

### **E. Instalación de PHP y MySQL**
```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php7.4 libapache2-mod-php7.4 php-mysql
```
*Instala PHP y su compatibilidad con Apache y MySQL.*

Verifica la versión de PHP instalada:
```bash
php -v
```

### **F. Instalación de Postfix**
```bash
sudo apt install postfix
```
Selecciona `Internet Site` y define el nombre del sistema de correo:
```bash
tso.localhost
```
Si necesitas reconfigurar Postfix:
```bash
sudo dpkg-reconfigure postfix
```

### **G. Instalación de Dovecot**
```bash
sudo apt install dovecot-imapd dovecot-pop3d
```
Reiniciar el servicio:
```bash
sudo service dovecot restart
```

### **H. Instalación de SquirrelMail**
```bash
cd /var/www/html/
sudo wget https://sourceforge.net/projects/squirrelmail/files/stable/1.4.22/squirrelmail-webmail-1.4.22.zip
sudo apt install unzip  # En caso de no tener instalado unzip
sudo unzip squirrelmail-webmail-1.4.22.zip
sudo mv squirrelmail-webmail-1.4.22 squirrelmail
sudo chown -R www-data:www-data /var/www/html/squirrelmail/
sudo chmod 755 -R /var/www/html/squirrelmail/
```

### **I. Configuración de SquirrelMail**
```bash
sudo perl /var/www/html/squirrelmail/config/conf.pl
```
Opciones:
1. Opción `2`, luego `1` y coloca el dominio.
2. Presiona `R` para regresar, selecciona `4` y modifica los siguientes valores:
   - **1:** `/var/www/html/squirrelmail/data/`
   - **2:** `/var/www/html/squirrelmail/attach/`
   - **11:** `true`
3. Guarda (`s`) y sal (`q`).

### **J. Creación de usuarios**
#### Usuario: gerencia
```bash
sudo adduser gerencia
sudo usermod -m -d /var/www/html/gerencia gerencia
sudo mkdir -p /var/www/html/gerencia
```
#### Usuario: sistemas
```bash
sudo adduser sistemas
sudo usermod -m -d /var/www/html/sistemas sistemas
sudo mkdir -p /var/www/html/sistemas
```

---
## **3. Acceso desde la Red Local**
1. Configurar el firewall para permitir tráfico en los puertos SMTP, IMAP y HTTP.
2. Verificar la dirección IP del servidor con `ip a`.
3. Probar el acceso desde un navegador con:
   ```
   http://[IP_DEL_SERVIDOR]/squirrelmail/
   ```

---
## **Mejoras Sugeridas**
  ✅ **Automatización:** Usar **Ansible** o scripts Bash.
  ✅ **Seguridad:** Implementar SSL/TLS y autenticación de dos factores.
  ✅ **Alternativas Modernas:** Considerar **Roundcube** en lugar de SquirrelMail para una mejor experiencia de usuario.
  ✅ **Monitoreo:** Configurar herramientas como **Prometheus** o **Grafana**.
  ✅ **Implementación en Contenedores:** Utilizar **LXD** o **Docker** para una infraestructura más flexible.

---
📌 **¡Listo! Tu servidor de correo en Ubuntu está funcionando. 🚀**
