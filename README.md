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

**Alternativa:** Ejemplo de cómo desplegar en un contenedor LXD (Leer documentación en https://canonical.com/lxd):
```bash
lxc launch ubuntu:20.04 servidor-correo
```
---
## **2. Configuración de Postfix y SquirrelMail**
### **A. Configuración del dominio local**
```bash
sudo nano /etc/hosts
```
*Ejemplo de entrada:*  
```
127.0.0.1   localhost
127.0.1.1   servidor-correo.local   servidor-correo
```

### **B. Verificación de herramientas esenciales**
```bash
if ! command -v ping &> /dev/null; then
    echo "iputils-ping no está instalado. Instalando..."
    sudo apt-get install -y iputils-ping
fi

if ! command -v ip &> /dev/null; then
    echo "iproute2 no está instalado. Instalando..."
    sudo apt-get install -y iproute2
fi
```

### **C. Actualización del sistema**
```bash
sudo apt update && sudo apt upgrade
```

### **D. Instalación de Apache**
```bash
sudo apt-get install apache2
```

### **E. Instalación de PHP y MySQL**
```bash
sudo apt install php libapache2-mod-php php-mysql
```

### **F. Instalación de Postfix**
```bash
sudo apt install postfix
```

### **G. Instalación de Dovecot**
```bash
sudo apt install dovecot-imapd dovecot-pop3d
```

### **H. Instalación de SquirrelMail**
```bash
cd /var/www/html/
sudo wget https://sourceforge.net/projects/squirrelmail/files/stable/1.4.22/squirrelmail-webmail-1.4.22.zip
sudo unzip squirrelmail-webmail-1.4.22.zip
sudo mv squirrelmail-webmail-1.4.22 squirrelmail
sudo chown -R www-data:www-data /var/www/html/squirrelmail/
sudo chmod 755 -R /var/www/html/squirrelmail/
```

### **I. Configuración de SquirrelMail**
```bash
sudo perl /var/www/html/squirrelmail/config/conf.pl
```

### **J. Creación de usuarios**
```bash
sudo adduser usuario1
```

---
## **3. Acceso desde la Red Local**
1. Configurar el firewall para permitir tráfico en los puertos SMTP, IMAP y HTTP.
2. Verificar la dirección IP del servidor con `ip a`.
3. Probar el acceso desde un navegador con `http://[IP_DEL_SERVIDOR]/squirrelmail/`.

---
## **Mejoras Sugeridas**
✅ **Automatización:** Usar **Ansible** o scripts Bash.  
✅ **Seguridad:** Implementar SSL/TLS y autenticación de dos factores.  
✅ **Alternativas Modernas:** Considerar **Roundcube** en lugar de SquirrelMail para una mejor experiencia de usuario.  
✅ **Monitoreo:** Configurar herramientas como **Prometheus** o **Grafana**.  
✅ **Implementación en Contenedores:** Utilizar **LXD** o **Docker** para una infraestructura más flexible.  

---
📌 **¡Listo! Tu servidor de correo en Ubuntu está funcionando. 🚀**
