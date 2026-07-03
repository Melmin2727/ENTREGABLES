# Inventario de Recursos AWS — Comercial Nova

## 1. Introducción

Este documento presenta el inventario completo de los recursos AWS desplegados para el proyecto, con fines de control, auditoría y mantenimiento. Se recomienda mantener este inventario actualizado ante cualquier cambio en la infraestructura.

## 2. Inventario General

| Nombre | Servicio | Estado | Región | Descripción | Uso |
|---|---|---|---|---|---|
| `vpc-comercial-nova` | Amazon VPC | Activo | us-east-1 | VPC principal, CIDR 10.0.0.0/16 | Aislamiento de red de todo el proyecto |
| `subnet-publica-a` | Subred | Activo | us-east-1a | CIDR 10.0.1.0/24, subred pública | Alojamiento del ALB |
| `subnet-publica-b` | Subred | Activo | us-east-1b | CIDR 10.0.2.0/24, subred pública | Alojamiento del ALB (segunda AZ) |
| `subnet-privada-app-a` | Subred | Activo | us-east-1a | CIDR 10.0.11.0/24, subred privada | Instancias EC2 de WordPress |
| `subnet-privada-app-b` | Subred | Activo | us-east-1b | CIDR 10.0.12.0/24, subred privada | Instancias EC2 de WordPress (segunda AZ) |
| `subnet-privada-datos-a` | Subred | Activo | us-east-1a | CIDR 10.0.21.0/24, subred privada | Instancia RDS |
| `subnet-privada-datos-b` | Subred | Activo | us-east-1b | CIDR 10.0.22.0/24, subred privada | Instancia RDS (subnet group Multi-AZ) |
| `igw-comercial-nova` | Internet Gateway | Activo | us-east-1 | Puerta de enlace a Internet | Tráfico saliente/entrante de subredes públicas |
| `sg-alb` | Security Group | Activo | us-east-1 | Reglas HTTP/HTTPS públicas | Firewall del ALB |
| `sg-ec2-wordpress` | Security Group | Activo | us-east-1 | Reglas HTTP desde ALB y SSH restringido | Firewall de instancias EC2 |
| `sg-rds` | Security Group | Activo | us-east-1 | Regla MySQL solo desde EC2 | Firewall de RDS |
| `wordpress-comercial-nova-01` | Amazon EC2 | Activo (gestionado por ASG) | us-east-1a | `t3.micro`, Ubuntu 22.04 | Servidor de aplicación WordPress 1 |
| `wordpress-comercial-nova-02` | Amazon EC2 | Activo (gestionado por ASG) | us-east-1b | `t3.micro`, Ubuntu 22.04 | Servidor de aplicación WordPress 2 |
| `wordpress-comercial-nova-ami` | Amazon Machine Image | Disponible | us-east-1 | AMI personalizada con Apache/PHP/WordPress | Base del Launch Template |
| `lt-wordpress-nova` | Launch Template | Activo | us-east-1 | Define configuración de instancias del ASG | Plantilla de lanzamiento del Auto Scaling |
| `asg-wordpress-nova` | Auto Scaling Group | Activo | us-east-1 | Min: 2, Max: 4, Deseado: 2 | Escalado automático de instancias de aplicación |
| `alb-comercial-nova` | Application Load Balancer | Activo | us-east-1 | Internet-facing, HTTP (HTTPS pendiente) | Distribución de tráfico entre instancias EC2 |
| `tg-wordpress-nova` | Target Group | Activo | us-east-1 | Health check en `/wp-login.php` | Registro de instancias saludables del ALB |
| `wordpress-db-comercial-nova` | Amazon RDS (MySQL 8.0) | Disponible | us-east-1 | `db.t3.micro`, Single-AZ, 20 GiB gp3 | Base de datos de WordPress |
| `subnet-group-rds-nova` | DB Subnet Group | Activo | us-east-1 | Subredes privadas de datos | Ubicación de red de RDS |
| `comercial-nova-media` | Amazon S3 | Activo | us-east-1 | Versionado habilitado, cifrado SSE-S3 | Almacenamiento de medios y respaldos |
| `rol-ec2-wordpress-nova` | AWS IAM Role | Activo | Global | Instance Profile con acceso a S3 y CloudWatch | Rol de servicio para instancias EC2 |
| `perfil-ec2-wordpress-nova` | IAM Instance Profile | Activo | Global | Asociado al rol anterior | Vinculación del rol a las instancias EC2 |
| `alerta-comercial-nova` | Amazon SNS Topic | Activo | us-east-1 | Suscripción por correo electrónico | Notificación de alarmas de CloudWatch |
| `dashboard-comercial-nova` | CloudWatch Dashboard | Activo | us-east-1 | Panel centralizado de métricas | Visualización operativa de la infraestructura |
| `alarma-cpu-alta-wordpress` | CloudWatch Alarm | Activo (OK) | us-east-1 | Umbral 80% CPU EC2 | Alerta de saturación de cómputo |
| `alarma-status-check-ec2` | CloudWatch Alarm | Activo (OK) | us-east-1 | Status check failed | Detección de fallas de instancia |
| `alarma-cpu-alta-rds` | CloudWatch Alarm | Activo (OK) | us-east-1 | Umbral 75% CPU RDS | Alerta de saturación de base de datos |
| `alarma-storage-bajo-rds` | CloudWatch Alarm | Activo (OK) | us-east-1 | Umbral 2 GiB libres | Alerta de espacio de almacenamiento bajo |

## 3. Resumen por Categoría de Servicio

| Categoría | Cantidad de recursos |
|---|---|
| Red (VPC, subredes, IGW) | 8 |
| Seguridad (Security Groups) | 3 |
| Cómputo (EC2, AMI, Launch Template, ASG) | 5 |
| Balanceo de carga (ALB, Target Group) | 2 |
| Base de datos (RDS, Subnet Group) | 2 |
| Almacenamiento (S3) | 1 |
| Identidad y accesos (IAM Role, Instance Profile) | 2 |
| Monitoreo (SNS, Dashboard, Alarmas) | 6 |
| **Total** | **29** |

## 4. Etiquetado (Tagging) Aplicado

Todos los recursos fueron etiquetados de forma consistente para facilitar su identificación, gestión de costos (Cost Allocation Tags) y automatización:

| Clave (Key) | Valor (Value) |
|---|---|
| `Project` | `ComercialNova` |
| `Environment` | `Academico` |
| `Owner` | `EquipoDevOps` |
| `ManagedBy` | `Manual / AWS-CLI` |

## 5. Conclusión

El inventario documentado permite tener trazabilidad completa de los 29 recursos AWS desplegados para el proyecto, facilitando tareas de auditoría, control de costos y eventual desmantelamiento ordenado (`terminate` / `delete`) de la infraestructura al finalizar el ciclo académico, evitando cargos residuales no planificados.
