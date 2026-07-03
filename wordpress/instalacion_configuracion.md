# Instalación y Configuración de WordPress sobre EC2 — Comercial Nova

## 1. Introducción

Este documento describe, paso a paso, el proceso real seguido para instalar y configurar WordPress sobre una instancia Amazon EC2 con Ubuntu Server, conectada a Amazon RDS (MySQL), como base para la posterior creación de la AMI utilizada por el Auto Scaling Group.

## 2. Creación de la Instancia EC2

1. Acceder a la consola de AWS → EC2 → **Launch Instance**.
2. Configurar:
   - **Nombre**: `wordpress-comercial-nova-01`
   - **AMI**: Ubuntu Server 22.04 LTS (HVM), SSD Volume Type
   - **Tipo de instancia**: `t3.micro` (nivel gratuito / ambiente académico)
   - **Par de claves**: `comercial-nova-key.pem` (generado y descargado previamente)
   - **Red**: VPC `vpc-comercial-nova`, subred privada de aplicación (AZ-a)
   - **Auto-asignar IP pública**: deshabilitado (el acceso se realiza a través del ALB o mediante bastión/SSM)
   - **Almacenamiento**: 20 GiB gp3
3. Asociar el **Security Group** `sg-ec2-wordpress` (ver [`../seguridad/matriz_accesos.md`](../seguridad/matriz_accesos.md)).
4. Lanzar la instancia.

> **Captura sugerida:** pantalla de configuración de "Launch Instance" mostrando AMI, tipo de instancia y VPC seleccionada.

## 3. Configuración de Security Groups

| Security Group | Puerto | Protocolo | Origen | Descripción |
|---|---|---|---|---|
| `sg-alb` | 80, 443 | TCP | 0.0.0.0/0 | Tráfico HTTP/HTTPS desde Internet |
| `sg-ec2-wordpress` | 80 | TCP | `sg-alb` | Tráfico solo desde el ALB |
| `sg-ec2-wordpress` | 22 | TCP | IP administrativa/bastión | Acceso SSH restringido |
| `sg-rds` | 3306 | TCP | `sg-ec2-wordpress` | Acceso a MySQL solo desde EC2 |

> **Captura sugerida:** reglas de entrada del Security Group `sg-ec2-wordpress` en la consola de AWS.

## 4. Instalación de Apache

Conectado por SSH a la instancia (a través de bastión o Session Manager):

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl status apache2
```

> **Captura sugerida:** salida de `systemctl status apache2` mostrando el servicio `active (running)`.

## 5. Instalación de PHP

WordPress requiere PHP y sus extensiones asociadas:

```bash
sudo apt install php php-mysql php-curl php-gd php-mbstring php-xml \
  php-xmlrpc php-soap php-intl php-zip libapache2-mod-php -y
sudo systemctl restart apache2
php -v
```

> **Captura sugerida:** salida de `php -v` confirmando la versión instalada (PHP 8.1+).

## 6. Instalación del Cliente MySQL

Para validar la conectividad hacia Amazon RDS desde la instancia EC2:

```bash
sudo apt install mysql-client -y
```

Prueba de conexión al endpoint de RDS:

```bash
mysql -h wordpress-db.xxxxxxxxxxxx.us-east-1.rds.amazonaws.com \
  -u admin_nova -p
```

> **Captura sugerida:** conexión exitosa al prompt `mysql>` desde la instancia EC2.

## 7. Descarga e Instalación de WordPress

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo cp -R wordpress/* /var/www/html/
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

> **Captura sugerida:** listado (`ls -la /var/www/html`) mostrando los archivos de WordPress copiados.

## 8. Configuración de `wp-config.php`

```bash
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

Parámetros configurados para la conexión con RDS:

```php
define( 'DB_NAME', 'wordpress_nova' );
define( 'DB_USER', 'admin_nova' );
define( 'DB_PASSWORD', '********' ); // gestionado de forma segura, no versionado en el repositorio
define( 'DB_HOST', 'wordpress-db.xxxxxxxxxxxx.us-east-1.rds.amazonaws.com' );
define( 'DB_CHARSET', 'utf8mb4' );
define( 'DB_COLLATE', '' );
```

> Las **claves de seguridad únicas** (`AUTH_KEY`, `SECURE_AUTH_KEY`, etc.) se generaron desde el servicio oficial de WordPress (`https://api.wordpress.org/secret-key/1.1/salt/`) y se reemplazaron en el archivo.

> **Captura sugerida:** fragmento de `wp-config.php` con los parámetros de conexión (credenciales ocultas).

## 9. Conexión con Amazon RDS

Antes de continuar, se creó la base de datos en la instancia RDS:

```sql
CREATE DATABASE wordpress_nova CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'admin_nova'@'%' IDENTIFIED BY '********';
GRANT ALL PRIVILEGES ON wordpress_nova.* TO 'admin_nova'@'%';
FLUSH PRIVILEGES;
```

> **Captura sugerida:** salida de `SHOW DATABASES;` confirmando la existencia de `wordpress_nova`.

## 10. Creación del Virtual Host de Apache

```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Contenido del Virtual Host:

```apacheconf
<VirtualHost *:80>
    ServerAdmin admin@comercialnova.com
    DocumentRoot /var/www/html
    ServerName wordpress-comercial-nova.local

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Activación del sitio y del módulo `rewrite` (necesario para los permalinks de WordPress):

```bash
sudo a2ensite wordpress.conf
sudo a2enmod rewrite
sudo a2dissite 000-default.conf
```

## 11. Permisos de Archivos

```bash
sudo chown -R www-data:www-data /var/www/html/
sudo find /var/www/html/ -type d -exec chmod 755 {} \;
sudo find /var/www/html/ -type f -exec chmod 644 {} \;
```

Estos permisos siguen la recomendación oficial de WordPress: directorios en `755` y archivos en `644`, con el usuario `www-data` (propietario del proceso de Apache) como dueño.

## 12. Reinicio de Apache

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```

## 13. Instalación Final (Asistente Web)

1. Acceder al DNS de la instancia (o del ALB una vez configurado) desde el navegador.
2. Seleccionar idioma **Español**.
3. Completar el formulario de instalación:
   - Título del sitio: **Comercial Nova**
   - Usuario administrador: `admin_nova`
   - Contraseña: generada de forma segura (no almacenada en el repositorio)
   - Correo electrónico: `contacto@comercialnova.com`
4. Finalizar la instalación y acceder a `wp-admin`.

> **Captura sugerida:** pantalla final de instalación de WordPress mostrando el mensaje "¡Éxito!".

## 14. Creación de la AMI para Auto Scaling

Una vez validada la instalación:

```bash
aws ec2 create-image \
  --instance-id i-0123456789abcdef0 \
  --name "wordpress-comercial-nova-ami" \
  --description "AMI base con Apache, PHP y WordPress configurado" \
  --no-reboot
```

Esta AMI se utilizó como base del **Launch Template** del Auto Scaling Group, garantizando que cada nueva instancia lanzada automáticamente cuente con la misma configuración validada.

## 15. Resumen del Proceso

| Paso | Estado |
|---|---|
| Creación de instancia EC2 | ✅ Completado |
| Configuración de Security Groups | ✅ Completado |
| Instalación de Apache | ✅ Completado |
| Instalación de PHP | ✅ Completado |
| Instalación de cliente MySQL | ✅ Completado |
| Descarga e instalación de WordPress | ✅ Completado |
| Configuración de `wp-config.php` | ✅ Completado |
| Conexión con RDS validada | ✅ Completado |
| Virtual Host configurado | ✅ Completado |
| Permisos aplicados | ✅ Completado |
| Instalación final vía navegador | ✅ Completado |
| Creación de AMI | ✅ Completado |
